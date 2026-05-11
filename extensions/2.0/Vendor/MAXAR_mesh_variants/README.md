# MAXAR_mesh_variants

## Contributors

* Erik Dahlström, Maxar, [@erikdahlstrom](https://github.com/erikdahlstrom)
* Sam Suhag, Cesium, [@sanjeetsuhag](https://github.com/sanjeetsuhag)
* Björn Blissing, Maxar [@bjornblissing](https://github.com/bjornblissing)

## Status

**Version 1.0.0**, November 4, 2025

## Dependencies

Written against the glTF 2.0 spec.

## Contents

- [Overview](#overview)
- [Variants](#variants)
- [Mappings](#mappings)
- [Validation and Constraints](#validation-and-constraints)
- [Example](#example)
- [Optional vs. Required](#optional-vs-required)
- [glTF Schema Updates](#gltf-schema-updates)

## Overview

This extension provides a compact representation for multiple mesh variants of an asset in glTF.

## Variants

For a glTF asset, a mesh `variant` represents a combination of meshes that are selected for rendering by a set of nodes based on _mappings_.

All available _variants_ are defined in the glTF root extension as an array of objects, each with a _name_ property.

The _default_ property is the index of the variant selected by default. The meshes mapped to the default must match the meshes initially selected by nodes for rendering, per vanilla glTF behavior.

## Mappings

For a given node, each entry in the mappings array represents a mesh that should be selected for rendering when one of its variants is active. Each entry in the mappings array is an object that specifies a mesh by its index in the root level `meshes` array and an array of variants, each by their respective indices in the root level `variants` array.

A variant may only be used once across all the entries in the mappings array.

When the active variant is referenced in a mapping, a compliant viewer will select its meshes for
rendering. Application-specific logic may allow the activation of different variants per-node, enabling different runtime configurations of the model.

## Validation and Constraints

This extension includes comprehensive validation to ensure data integrity:

- **String Validation**: Variant and mapping names must be non-empty (minLength: 1) and reasonably sized (maxLength: 256)
- **Index Validation**: All indices must be non-negative integers (minimum: 0)
- **Array Limits**: Arrays are limited to reasonable sizes (maxItems: 256) to prevent performance issues
- **Unique Constraints**: Variant indices in mappings must be unique within each mapping

## Example

_This section is non-normative._

The following example illustrates the representation of a car model with varying states of damage applied to the model.

| Variants               |
| ---------------------- |
| `Pristine` - _default_ |
| `Damaged`              |
| `Destroyed`            |

At the root level, this will be described as follows:

```json
{
  "asset": {"version": "2.0"},
  "extensions": {
    "MAXAR_mesh_variants": {
      "default": 0,
      "variants": [
        {"name": "Pristine"  },
        {"name": "Damaged"   },
        {"name": "Destroyed" },
      ]
    }
  }
}
```

For the purposes of illustration, let's consider that the model consists of 3 nodes - body, wheels and lights.

```json
"nodes": [
  {
    "name": "Car Body",
    "mesh": 0,
    "extensions": {
      "MAXAR_mesh_variants" : {
        "mappings": [
          {
            "mesh": 0,
            "variants": [0]
          },
          {
            "mesh": 1,
            "variants": [1]
          },
          {
            "mesh": 2,
            "variants": [2]
          }
        ],
      }
    }
  },
  {
    "name": "Car Wheels",
    "mesh": 3,
    "extensions": {
      "MAXAR_mesh_variants" : {
        "mappings": [
          {
            "mesh": 3,
            "variants": [0]
          },
          {
            "mesh": 4,
            "variants": [1]
          },
          {
            "mesh": 5,
            "variants": [2]
          }
        ],
      }
    }
  },
  {
    "name": "Car Lights",
    "mesh": 6,
    "extensions": {
      "MAXAR_mesh_variants" : {
        "mappings": [
          {
            "mesh": 6,
            "variants": [0]
          },
          {
            "mesh": 7,
            "variants": [1, 2]
          },
        ],
      }
    }
  }
]
```

## Optional vs. Required

This extension is considered optional, meaning it should be placed in the glTF root's `extensionsUsed` list, but not in the `extensionsRequired` list.

## glTF Schema Updates

- **glTF extension JSON schema**: [glTF.MAXAR_mesh_variants.schema.json](./schema/glTF.MAXAR_mesh_variants.schema.json)
- **glTF node extension JSON schema**: [node.MAXAR_mesh_variants.schema.json](./schema/node.MAXAR_mesh_variants.schema.json)
