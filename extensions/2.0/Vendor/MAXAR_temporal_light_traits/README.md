# MAXAR_temporal_light_traits

## Contributors

* Richard Moreland, Maxar
* Björn Blissing, Maxar, [@bjornblissing](https://github.com/bjornblissing)

## Status

**Version 1.0.0**, April 28, 2026

## Dependencies

Written against the glTF 2.0 spec.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY in this document are to be interpreted as described in RFC 2119.

## Contents

- [Overview](#overview)
- [Optional vs Required](#optional-vs-required)
- [Extending Lights with Temporal Traits](#extending-lights-with-temporal-traits)
- [Trait Types](#trait-types)
  - [Flashing](#flashing)
    - [Properties](#properties)
    - [Waveform Types](#waveform-types)
- [Examples](#examples)
- [Schema](#schema)


## Overview

This extension defines a set of traits to describe the animated aspects of a light. It builds upon the `KHR_lights_punctual` extension's description of a light, adding temporal behaviors that can be used to create dynamic lighting effects such as flashing navigation lights, strobing emergency beacons, or pulsing ambient lighting.

The extension is designed to be lightweight and efficient, providing mathematical descriptions of common lighting patterns that can be computed in real-time without requiring complex animation data.

## Optional vs Required

This extension is optional, meaning it should be placed in the glTF root's `extensionsUsed` list, but not in the `extensionsRequired` list.

Clients that do not implement this extension SHOULD render the light using its static `KHR_lights_punctual` definition. Clients that implement this extension MUST apply the temporal traits as a visual enhancement on top of that static definition.

## Extending Lights with Temporal Traits

This extension augments lights defined by `KHR_lights_punctual` with parametric temporal behavior. Instead of storing sampled animation data, it describes how a light changes over time using a small set of properties that implementations can evaluate at runtime.

The following sample shows how `KHR_lights_punctual` defines a light and `MAXAR_temporal_light_traits` adds a `flashing` trait to that light, together creating a flashing navigation light:

```json
{
  "extensionsUsed": ["KHR_lights_punctual", "MAXAR_temporal_light_traits"],
  "extensions": {
    "KHR_lights_punctual": {
      "lights": [
        {
          "type": "point",
          "color": [1.0, 0.0, 0.0],
          "intensity": 2.0,
          "range": 100.0,
          "name": "Navigation Light - Port",
          "extensions": {
            "MAXAR_temporal_light_traits": {
              "flashing": {
                "frequency": 1.0,
                "waveform": "square",
                "duty": 0.75,
                "amplitudeScale": 1.0,
                "amplitudeOffset": 0.0,
                "phaseOffset": 0.0
              }
            }
          }
        }
      ]
    }
  }
}
```

## Trait Types

A light MAY omit this extension entirely, in which case the light does not change with time. A light MAY use one of the traits defined below. Currently, only the `flashing` trait is defined, but the extension is designed to be extensible for future trait types such as more complex flashing patterns, color cycling, or morse code patterns.

### Flashing

The `"flashing"` trait describes a simple flashing pattern that can be characterized by a waveform type and frequency. When this trait is applied to a light, an implementation MUST modulate the light's intensity over time according to the following mathematical function:

```
intensity(t) = baseIntensity * clamp(amplitudeOffset + amplitudeScale * waveform((t + phaseOffset) * frequency), 0.0, 1.0)
```

Where:
- `baseIntensity` is the light's original intensity from `KHR_lights_punctual`
- `t` is the current time in seconds
- `waveform()` is the selected waveform function normalized to range [0.0, 1.0]
- `clamp()` ensures the final multiplier stays within valid bounds

#### Properties

| Property      | Type | Description | Required | Default |
|:--------------|:-----|:------------|:---------|:--------|
| `waveform`    | string | Shape of waveform: `"sine"`, `"square"`, or `"triangle"` | **Yes** | - |
| `frequency`   | number | Frequency of the waveform in Hertz (Hz). MUST be positive. | **Yes** | - |
| `duty`        | number | Portion of period the light is on as normalized value [0.0, 1.0]. MUST only be specified when `waveform` is `"square"`. | No | `0.5` |
| `amplitudeScale` | number | Scale factor applied to waveform output. MUST be non-negative. | No | `1.0` |
| `amplitudeOffset` | number | Offset added to waveform output after scaling. MAY be negative. | No | `0.0` |
| `phaseOffset` | number | Phase offset in seconds to shift the waveform in time. | No | `0.0` |

#### Waveform Types

Each waveform function takes the cycle position `u = (t + phaseOffset) * frequency` as input and is evaluated using the fractional part of `u`:

```
p = u - floor(u)    // p in [0, 1)
```

All waveforms are normalized to output in the range `[0, 1]`.

- **`"sine"`**: Smooth sinusoidal oscillation. Creates gentle, natural-looking intensity changes.
  ```
  sine(u) = (1 - cos(2 * pi * p)) / 2
  ```
  Output starts at 0 at the cycle boundary, rises smoothly to 1 at `p = 0.5`, and returns to 0 at the end of the cycle.

- **`"square"`**: Sharp on/off transitions. The `duty` property `d` controls the portion of the cycle the light is on.
  ```
  square(u) = 1    if p < d
  square(u) = 0    if p >= d
  ```
  Output is on (1) for the first `d` portion of each cycle and off (0) for the remainder.

- **`"triangle"`**: Linear ramp up and down with equal rise and fall times.
  ```
  triangle(u) = 2 * p          if p < 0.5
  triangle(u) = 2 * (1 - p)    if p >= 0.5
  ```
  Output starts at 0 at the cycle boundary, rises linearly to 1 at `p = 0.5`, and falls linearly back to 0 at the end of the cycle.

## Examples

**Basic Strobe Light (1 Hz square wave):**
```json
"flashing": {
  "frequency": 1.0,
  "waveform": "square",
  "duty": 0.1
}
```

**Gentle Pulsing Light (0.5 Hz sine wave):**
```json
"flashing": {
  "frequency": 0.5,
  "waveform": "sine",
  "amplitudeScale": 0.8,
  "amplitudeOffset": 0.2
}
```

**Synchronized Navigation Lights (same frequency, different phase):**
```json
// Port light (red)
"flashing": {
  "frequency": 2.0,
  "waveform": "square",
  "duty": 0.5,
  "phaseOffset": 0.0
}

// Starboard light (green) - 180° out of phase
"flashing": {
  "frequency": 2.0,
  "waveform": "square",
  "duty": 0.5,
  "phaseOffset": 0.25
}
```

**Emergency Beacon (fast triangle wave):**
```json
"flashing": {
  "frequency": 4.0,
  "waveform": "triangle",
  "amplitudeScale": 1.2,
  "amplitudeOffset": -0.1
}
```

## Schema

* [light.MAXAR_temporal_light_traits.schema.json](schema/light.MAXAR_temporal_light_traits.schema.json)