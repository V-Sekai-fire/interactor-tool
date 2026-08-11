# Tools

A tool is a name, a JSON schema, and a function. Adding one is a row here and a function in
`src/`, not a repository and not a deploy.

## The transport

MCP frames go over iceoryx2, not over a socket. The protocol is JSON-RPC and does not care,
and this keeps the plane free of networking, which is what makes it a plane rather than an
edge.

The payloads are the reason it is worth doing rather than a technicality. A tool call carries
a mesh, a motion, or a stage: `100STYLE` alone unpacks to 3.2 GB, and a single generated clip
is a megabyte. Over HTTP each call is a serialise, a copy, and a parse. Over the bus the
caller writes the bytes once and the tool reads the same bytes.

## The rule a tool obeys

**A tool is stateless between calls.** It takes arguments, does work, writes a file, and
returns a path. If something needs to persist between calls it is not a tool, it is a plane,
and it belongs in its own repository with its own tick.

**A tool emits USD.** Every converter in this project already does: the SOMA rig, generated
motion, 100STYLE, the Fab clips, and the CSG props. The edge does not introduce a format and
does not accept one that states no unit.

**A tool measures what it produced.** Four separate unit faults in this project were caught by
one habit: refuse the result if it is not the size it claims to be. A skeleton that is not
between 0.5 and 2.5 m tall is not a person whatever the header said, and a prop over 4 m is
not furniture.

## The tools

| name | in | out |
| --- | --- | --- |
| `csg_build` | solid list, dimensions from standards | glTF |
| `gltf_to_usd` | glTF | USD, metres, dimensions verified |
| `bvh_to_usd` | BVH | USD, unit chosen by measuring the skeleton |
| `motion_gen` | text segments, durations, constraints | SOMA npz and USD |
| `render_check` | motion | contact sheet, for looking at it |

## Why the surface is generic

The Godot MCP addon exposes `create_node`, `call_method`, and `call_singleton`. That was
enough to build five pieces of furniture and export them, without one Godot-specific
furniture command being designed. A narrow generic surface outlived a wide specific one, and
the same applies here: a tool that takes a CSG tree beats five tools that each make a chair.
