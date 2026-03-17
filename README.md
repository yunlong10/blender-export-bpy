# Blender Export BPY Code

A Blender addon that exports your scene as a reproducible Python (`bpy`) script. Run the exported script in any Blender instance to reconstruct the exact same scene.

![Blender 3.6+](https://img.shields.io/badge/Blender-3.6%2B-orange)
![License](https://img.shields.io/badge/License-MIT-green)

## Why?

Blender can export to FBX, OBJ, glTF, USD — but none of these export as **Python code**. This addon fills that gap.

Use cases:

- **Benchmarking**: Compare AI-generated scene code against human-authored ground truth using the same `bpy` API representation
- **Reproducibility**: Share scenes as self-contained Python scripts, no `.blend` files needed
- **Education**: See exactly what `bpy` calls produce a given scene
- **Version control**: Track scene changes as readable Python diffs
- **Automation**: Generate base scripts to modify programmatically

## Installation

1. Download `export_bpy_code.py`
2. In Blender: `Edit → Preferences → Add-ons → Install...`
3. Select the downloaded file
4. Enable the checkbox next to "Export Scene as BPY Code"

## Usage

`File → Export → Blender Python Script (.py)`

### Options

| Option | Default | Description |
|--------|---------|-------------|
| Include Clear Scene | ✅ | Prepend code to wipe the scene before rebuilding |
| Include Camera | ✅ | Export camera objects |
| Include Lights | ✅ | Export light objects |
| Include Render Settings | ❌ | Export render engine, resolution, samples |
| Precision | 6 digits | Decimal places for coordinates (4 / 6 / 8) |

### Running the exported script

```bash
# In Blender's Script Editor: open and run the .py file

# Or from command line:
blender --python my_scene.py

# Or headless (no GUI):
blender --background --python my_scene.py
```

## What gets exported

| Element | How it's exported |
|---------|-------------------|
| Cubes (8 verts, 6 faces) | `bpy.ops.mesh.primitive_cube_add()` |
| Planes (4 verts, 1 face) | `bpy.ops.mesh.primitive_plane_add()` |
| Cylinders (32-side) | `bpy.ops.mesh.primitive_cylinder_add()` |
| Spheres (UV sphere-like) | `bpy.ops.mesh.primitive_uv_sphere_add()` |
| Complex meshes | Cube fallback + comment (see Limitations) |
| Materials | Principled BSDF parameters (color, roughness, metallic, emission, alpha) |
| Lights | Type, energy, color, size, spot angle |
| Cameras | Lens, clip range, sensor width |
| Empties | Display type and size |
| Parent relationships | `obj.parent = ...` |
| Render settings | Engine, resolution, samples (optional) |

## Limitations

- **Complex meshes**: Meshes that don't match a known primitive (cube, plane, cylinder, sphere) are exported as a cube with a comment. For exact reconstruction of complex geometry, consider pairing this with `.obj` or `.fbx` exports for those specific objects.
- **Modifiers**: Not exported. Apply modifiers before exporting if needed.
- **Textures/Images**: Image textures are not exported — only solid Principled BSDF parameters.
- **Node graphs**: Only the Principled BSDF node is read. Complex shader node setups are simplified.
- **Constraints & Drivers**: Not exported.
- **Shape keys / Armatures**: Not exported.
- **Collections**: Object hierarchy via parenting is exported, but Blender collections are not.

## Example output

```python
import bpy
import math

# Clear Scene
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)
# ...

# Materials
mat_Wall = bpy.data.materials.new(name='Wall')
mat_Wall.use_nodes = True
_bsdf = mat_Wall.node_tree.nodes['Principled BSDF']
_bsdf.inputs['Base Color'].default_value = (0.95, 0.92, 0.87, 1.0)
_bsdf.inputs['Roughness'].default_value = 0.9
_bsdf.inputs['Metallic'].default_value = 0.0

# Objects
bpy.ops.mesh.primitive_cube_add(size=1, location=(-2.5, -2.0, 0.4))
obj = bpy.context.active_object
obj.name = 'Wall_Front'
obj.rotation_euler = (0.0, 0.0, 0.0)
obj.scale = (6.0, 0.08, 2.6)
obj.data.materials.append(bpy.data.materials['Wall'])
```

## Contributing

PRs welcome! Some ideas:

- [ ] Export modifiers as `bpy.ops` calls
- [ ] Export animation keyframes
- [ ] Support Geometry Nodes parameters
- [ ] Better complex mesh handling (export as bmesh construction)
- [ ] Export collections
- [ ] Import: reconstruct scene from exported `.py` (round-trip validation)

## License

MIT