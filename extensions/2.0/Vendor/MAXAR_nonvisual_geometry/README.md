# MAXAR_nonvisual_geometry

## Contributors

* Richard Mooreland
* Björn Blissing, [@bjornblissing](https://github.com/bjornblissing)

## Status

**Version 1.1.0**, July 4, 2025

## Dependencies

Written against the glTF 2.0 spec.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY in this document are to be interpreted as described in RFC 2119.

## Contents

- [Overview](#overview)
- [Shape of nonvisual geometry](#shape-of-nonvisual-geometry)
- [Types of Nonvisual Meshes](#types-of-nonvisual-meshes)
- [Adding Nonvisual Meshes](#adding-nonvisual-meshes)
- [Validation Rules and Constraints](#validation-rules-and-constraints)
- [Optional vs. Required](#optional-vs-required)
- [Implementation Notes](#implementation-notes)
- [Schema](#schema)

## Overview

In gaming and simulation engines, high resolution 3D models are often paired
with nonvisual versions of the model that are optimized for other purposes than
rendering and visualization. This extension to glTF enables designation of
meshes as nonvisual representations of nodes.  Each primitive of the nonvisual
mesh specifies its intended usage.

## Shape of nonvisual geometry

| Shape        | Description | Allowed `mesh.primitive.mode` |
| ------------ | ----------- | ------------------------------|
| `points`     | One or more points. | `0` - POINTS |
| `path`       | One polyline, which MAY be self-intersecting. | `1` - LINES, `2` - LINE_LOOP, `3` - LINE_STRIP _(for mode `1` only two vertex points are allowed)_.|
| `surface`    | One non-self intersecting triangulated surface. | `4` - TRIANGLES, `5` - TRIANGLE_STRIP, `6` - TRIANGLE_FAN  |
| `volume`     | One closed, convex, triangulated polyhedron. | `4` - TRIANGLES, `5` - TRIANGLE_STRIP |

## Types of Nonvisual Meshes

The `type` property is a free-text field that allows applications to define their own semantic meanings. However, well-known values SHOULD be used when possible. Commonly used types include:

| Type        | Description | Typical Shapes |
| ----------- | ----------- | -------------- |
| `collision` | Geometry used for collision detection and simulation | `surface`, `volume` |
| `roadway`   | Walkable or drivable surface | `surface` |
| `ladder`    | One line to convey where a player may enter/exit a ladder | `path` |
| `action`    | Geometry used to trigger the articulations assigned to this node | `surface`, `volume` |
| `trigger`   | Geometry used to trigger events or interactions | `surface`, `volume`, `points` |
| `navigation` | Geometry used for pathfinding and AI navigation | `surface`, `path` |
| `occlusion` | Geometry used for occlusion culling optimization | `surface`, `volume` |


## Adding Nonvisual Meshes

The following example illustrates how a node can designate a mesh as container for the nonvisual representation. This mesh can contain several different nonvisual representations of the mesh:

```json
"nodes": [{
    "name": "Detailed Building",
    "mesh": 0,
    "extensions": {
      "MAXAR_nonvisual_geometry": {
        "mesh": 1
      }
    }
  }
],
"meshes": [{
    "name": "Detailed Building Mesh",
    "primitives": [{
        "attributes": {
          "POSITION": 0,
          "NORMAL": 1
        },
        "indices": 2
      }
    ]
  }, {
    "name": "Nonvisual Building Mesh",
    "primitives": [{
        "attributes": {
          "POSITION": 3
        },
        "indices": 4,
        "extensions": {
          "MAXAR_nonvisual_geometry": {
            "shape": "surface",
            "type": "roadway"
          }
        }
      }, {
        "attributes": {
          "POSITION": 5
        },
        "indices": 6,
        "mode": 1,
        "extensions": {
          "MAXAR_nonvisual_geometry": {
            "shape": "path",
            "type": "ladder"
          }
        }
      }, {
        "attributes": {
          "POSITION": 7
        },
        "indices": 8,
        "extensions": {
          "MAXAR_nonvisual_geometry": {
            "shape": "volume",
            "type": "collision"
          }
        }
      }
    ]
  }
]
```

### Additional Examples

#### Simple Collision Volume

```json
{
  "nodes": [{
    "name": "Complex Machinery",
    "mesh": 0,
    "extensions": {
      "MAXAR_nonvisual_geometry": {
        "mesh": 1
      }
    }
  }],
  "meshes": [{
    "name": "Detailed Machinery Mesh",
    "primitives": [{
      "attributes": { "POSITION": 0, "NORMAL": 1, "TEXCOORD_0": 2 },
      "indices": 3
    }]
  }, {
    "name": "Collision Mesh",
    "primitives": [{
      "attributes": { "POSITION": 4 },
      "indices": 5,
      "mode": 4,
      "extensions": {
        "MAXAR_nonvisual_geometry": {
          "shape": "volume",
          "type": "collision"
        }
      }
    }]
  }]
}
```

#### Navigation and Trigger Points

```json
{
  "meshes": [{
    "name": "Navigation Mesh",
    "primitives": [{
      "attributes": { "POSITION": 0 },
      "indices": 1,
      "mode": 4,
      "extensions": {
        "MAXAR_nonvisual_geometry": {
          "shape": "surface",
          "type": "navigation"
        }
      }
    }, {
      "attributes": { "POSITION": 2 },
      "mode": 0,
      "extensions": {
        "MAXAR_nonvisual_geometry": {
          "shape": "points",
          "type": "trigger"
        }
      }
    }]
  }]
}
```

## Validation Rules and Constraints

- **Volume shapes** MUST represent closed, convex polyhedra
- **Surface shapes** MUST be non-self-intersecting
- **Path shapes** MAY be self-intersecting
- Nonvisual meshes SHOULD use minimal geometry optimized for their intended purpose
- Nonvisual meshes SHOULD NOT include materials, textures, or other rendering-related properties
- The `type` property MUST be a non-empty string. Applications MAY define custom semantic meanings, though well-known values SHOULD be used
- For each primitive carrying this extension, the primitive's `mode` MUST be one of the values listed for its `shape` in the "Shape of nonvisual geometry" table above. This relationship is not enforced by the JSON Schema, since an extension schema cannot constrain its parent primitive; producers MUST emit a compatible `mode`, and consumers and validators SHOULD reject incompatible combinations

## Optional vs. Required

This extension is considered optional, meaning it should be placed in the glTF root's `extensionsUsed` list, but not in the `extensionsRequired` list.

## Implementation Notes

### Best Practices

1. **Mesh Organization**: Keep nonvisual meshes separate from visual meshes. Use descriptive names to clearly identify the purpose of each nonvisual mesh.

2. **Geometry Optimization**: Nonvisual meshes SHOULD use the minimum geometry required for their purpose:
   - Collision volumes SHOULD be simplified convex hulls
   - Navigation surfaces SHOULD be low-poly representations
   - Trigger points SHOULD use single vertices when possible

3. **Performance Considerations**:
   - Nonvisual meshes are typically not rendered to the screen but may be processed by physics engines, AI systems, or other runtime components
   - Keep vertex counts low to minimize memory usage and processing overhead
   - Use appropriate primitive modes for the intended shape

### Common Pitfalls

1. **Mode Mismatch**: Ensure the primitive `mode` is compatible with the specified `shape`. This relationship is normative but is not enforced by the JSON Schema; producers and validators are responsible for checking it.

2. **Overcomplicated Geometry**: Avoid using high-resolution geometry for nonvisual purposes. Simple shapes are usually sufficient and more performant.

3. **Missing Required Properties**: Both `shape` and `type` are required properties in the mesh primitive extension.

### Runtime Behavior

- Applications MAY process nonvisual meshes; when they do, they MUST interpret each primitive according to its `type` and `shape`
- Nonvisual meshes are typically not rendered to the screen, except in debug views or development tools
- The extension allows multiple nonvisual representations per node, enabling complex interaction scenarios

## Schema

- **glTF node extension JSON schema**: [node.MAXAR_nonvisual_geometry.schema.json](schema/node.MAXAR_nonvisual_geometry.schema.json)
- **glTF mesh primitive extension JSON schema**: [mesh.primitive.MAXAR_nonvisual_geometry.schema.json](schema/mesh.primitive.MAXAR_nonvisual_geometry.schema.json)
