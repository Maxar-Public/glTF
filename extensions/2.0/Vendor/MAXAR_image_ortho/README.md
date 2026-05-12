# MAXAR_image_ortho

## Contributors

* Erik Dahlström, [@erikdahlstrom](https://github.com/erikdahlstrom)
* Johan Bejeryd
* Johan Borg, [@jo-borg](https://github.com/jo-borg)
* Björn Blissing, [@bjornblissing](https://github.com/bjornblissing)

## Status

**Version 1.0.0**, November 3, 2025

## Dependencies

Written against the glTF 2.0 spec.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY in this document are to be interpreted as described in RFC 2119.

## Overview

This extension allows specifying that an image is orthographic, and which coordinate reference system (CRS) was used to project it.

The ability to know that an image is orthographically projected can simplify some steps when reprocessing of the dataset is needed.

The extension consists of a transformation which is used to convert from pixel coordinates to world coordinates and the spatial reference system that was used to project the image.

The transformation is defined using a 2x3 affine transform matrix that operates on pixel coordinates in the image.

## Optional vs. Required

This extension is considered optional, meaning it should be placed in the glTF root's `extensionsUsed` list, but not in the `extensionsRequired` list.

## Compatibility and graceful degradation

Viewers that do not implement this extension will still load the asset normally. The extension only adds metadata. If it is ignored, images are displayed as standard glTF images without applying the orthographic transform or spatial reference; geographic positioning is omitted, but rendering remains valid.

## Transform Matrix

The 2x3 affine transformation matrix converts pixel coordinates to world coordinates. The matrix elements are stored as a 6-element array: `[a, b, c, d, e, f]`, which represents the transformation matrix:

```
[a c e]
[b d f]
[0 0 1]
```

Pixel to world transformation is performed using the following formula:

```
[X_world]   [a c e]   [X_pixel]
[Y_world] = [b d f] * [Y_pixel]
[   1   ]   [0 0 1]   [   1   ]
```

This can also be expressed as:

```
X_world = X_pixel * a + Y_pixel * c + e
Y_world = X_pixel * b + Y_pixel * d + f
```

Where:
- `X_pixel`, `Y_pixel` are the pixel coordinates in the image
- `X_world`, `Y_world` are the corresponding world coordinates in the specified spatial reference system
- `a`, `b`, `c`, `d`, `e`, `f` are the transform matrix elements stored as `[a, b, c, d, e, f]`
- Elements `a` and `d` represent scaling, `b` and `c` represent rotation/skew, `e` and `f` represent translation

All six matrix elements MUST be finite JSON numbers. The values `null`, `NaN`, `Infinity`, and `-Infinity` MUST NOT be used.

### Pixel Coordinate Convention

Pixel coordinates `(X_pixel, Y_pixel)` are continuous values relative to
the image's pixel grid:

- The origin `(0, 0)` is the center of the upper-left pixel.
- `X_pixel` increases to the right; `Y_pixel` increases downward.
- The pixel at column `i`, row `j` has its center at `(i, j)`.
- For an image with `width` columns and `height` rows, the upper-left
  corner of the image is at `(-0.5, -0.5)` and the lower-right corner is
  at `(width - 0.5, height - 0.5)`.

Implementations MUST use this convention when applying the `transform`
matrix; producers MUST compute matrix elements `a`–`f` consistently with
it.

### World Coordinate Axis Mapping

Implementations MUST interpret the two world components produced by the
`transform`, `X_world` and `Y_world`, according to the SRS `axis`
property:

- When `axis` is `"NED"`, `X_world` is the North component and `Y_world`
  is the East component.
- When `axis` is `"ENU"`, `X_world` is the East component and `Y_world`
  is the North component.

The vertical (Down or Up) component is not represented in the 2D
`transform`.

## Usage

This extension is applied to individual `image` objects within a glTF file to specify:

1. **Orthographic Projection**: Indicates that the image represents an orthographically projected view
2. **Spatial Reference**: Defines the coordinate reference system used for the projection
3. **Geometric Transformation**: Provides the affine transformation to convert between pixel and world coordinates

### Use Cases

- **Aerial/Satellite Imagery**: Georeferenced orthophotos with known coordinate systems
- **Map Overlays**: Precisely positioned map tiles or imagery layers
- **GIS Integration**: Images that need to be accurately positioned in geographic information systems
- **Reprocessing Workflows**: Scenarios where knowing the original projection simplifies data processing

Only the transformations defined on the image itself are necessary to visualize the image at its correct position. The image MAY be used by the glTF scene as well, but in scenarios where only the image is needed, this enables optimizing those operations.

## Spatial Reference Systems

This extension supports the following coordinate systems:

### Supported Coordinate Systems

| **System** | **Code** | **Description** | **Units** |
|:-----------|:---------|:----------------|:----------|
| **Geodetic** | `GEOD` | Geographic coordinates (latitude/longitude) | Degrees or Meters |
| **S2** | `S2F0` to `S2F5` | S2 geometry projection on faces 0-5 | Meters |
| **ECEF** | `ECEF` | Earth-Centered, Earth-Fixed (Geocentric) | Meters |
| **UTM** | `UTM##N/S` | Universal Transverse Mercator zones 01-60 | Meters |

### Reference Systems

- **ITRF2008**: International Terrestrial Reference Frame 2008
- **WGS84-G1762**: World Geodetic System 1984, realization G1762



### Defaults for GEOD and S2

For GEOD and S2 coordinate systems, `referenceSystem` and `epoch` MAY be
omitted. If omitted, implementations MUST assume `referenceSystem: "ITRF2008"`
and `epoch: "2005.0"`. If `referenceSystem` and `epoch` are provided for GEOD
or S2, they MUST be `"ITRF2008"` and `"2005.0"` respectively. The JSON Schema
encodes this via a conditional with defaults; for all other coordinate
systems, `referenceSystem` and `epoch` MUST be present.


## Example

### UTM Coordinate System

The following example shows an orthographic image using UTM Zone 14 North coordinates:

```json
{
  "extensionsUsed": [
    "MAXAR_image_ortho"
  ],
  "images": [{
      "bufferView": 0,
      "mimeType": "image/jpeg",
      "extensions": {
        "MAXAR_image_ortho": {
          "transform": [0.0, 8.1875, -8.1875, 0.0, 3542945.276954, 782027.09375],
          "srs": {
            "referenceSystem": "ITRF2008",
            "epoch": "2005.0",
            "coordinateSystem": "UTM14N",
            "elevation": "ELLIPSOID",
            "axis": "NED",
            "unitHorizontal": "METER",
            "unitVertical": "METER"
          }
        }
      }
    }
  ]
}
```

This transform matrix `[0.0, 8.1875, -8.1875, 0.0, 3542945.276954, 782027.09375]` represents:

- **Resolution**: 8.1875 meters per pixel along both pixel axes
- **Rotation**: The off-diagonal layout (`a = 0`, `b = 8.1875`, `c = -8.1875`, `d = 0`) encodes a 90° rotation: pixel +X maps to world +Y (east), and pixel +Y maps to world -X (south)
- **Origin**: Pixel (0,0) maps to UTM coordinates (3,542,945.28 N, 782,027.09 E)
- **Geographic location**: Central United States (UTM Zone 14 North) - 31.987513° N, 96.015177° W

The transformation converts pixel coordinates to world coordinates as:
```
X_world = X_pixel * 0.0     + Y_pixel * (-8.1875) + 3542945.276954
Y_world = X_pixel * 8.1875  + Y_pixel *   0.0     + 782027.09375
```

## Extension Properties

### MAXAR_image_ortho Object

| **Property** | **Type** | **Description** | **Required** | **Default** |
|:-------------|:---------|:----------------|:-------------|:------------|
| **transform** | `number[6]` | 2x3 affine transformation matrix for pixel-to-world coordinate conversion, stored as `[a, b, c, d, e, f]` representing matrix `[[a, c, e], [b, d, f]]` | No | `[1.0, 0.0, 0.0, 1.0, 0.0, 0.0]` |
| **srs** | `object` | Spatial reference system definition (see SRS Object below) |  Yes | - |

### SRS Object

| **Property** | **Type** | **Description** | **Required** | **Default** |
|:-------------|:---------|:----------------|:-------------|:------------|
| **referenceSystem** | `string` | Geodetic reference frame: `"ITRF2008"` or `"WGS84-G1762"` | Conditional | - |
| **epoch** | `string` | Coordinate epoch as decimal year (e.g., `"2005.0"`) | Conditional | - |
| **coordinateSystem** | `string` | Coordinate system: `"GEOD"`, `"ECEF"`, `"UTM##N/S"`, or `"S2F#"` | Yes | - |
| **elevation** | `string` | Vertical datum: `"ELLIPSOID"` or `"EGM2008"` | Yes | - |
| **axis** | `string` | Axis orientation: `"NED"` (North-East-Down) or `"ENU"` (East-North-Up) | No | `"NED"` |
| **unitHorizontal** | `string` | Horizontal coordinate units: `"METER"` or `"DEGREE"` | No | `"METER"` |
| **unitVertical** | `string` | Vertical coordinate units: `"METER"` | No | `"METER"` |

## JSON Schema

The extension is defined by the following JSON Schema files:

* **[image.MAXAR_image_ortho.schema.json](schema/image.MAXAR_image_ortho.schema.json)** - Main extension schema for image objects
* **[srs.MAXAR_image_ortho.schema.json](schema/srs.MAXAR_image_ortho.schema.json)** - Spatial reference system schema

