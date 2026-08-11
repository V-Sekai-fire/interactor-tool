# fabric-tool-edge

One edge that runs tools, so a new capability is a registered tool and not a new container.

## Why this is an edge and not a plane

`CLAUDE.md` settles it rather than leaving it to taste:

> A plane has no networking. An edge is a plane with networking. This is a definition, not a
> default, so there is no exception to check.

MCP is a wire protocol. Anything that speaks it has networking, so this is an edge. It obeys
every plane rule and adds one capability, the network. It holds no authority, runs no
simulation, and keeps no durable state.

## Why one edge and not one per tool

Before this, each capability wanted its own repository, its own container, and its own Fly
app: one to build props, one to convert motion, one to generate. Each is a deploy, an image
push, and a cold start, for a job that runs for seconds.

A tool is not a plane. It has no tick, holds no state between calls, and does not need a
core to itself. So the shape that fits is one edge with a tool table, and adding a capability
is adding a row.

## How a tool reaches a library

The same way the harness reaches iceoryx2, because weft already made this decision once.
`fabric-harness/iceoryx2.sigs` lists the C ABI it calls, `generate_stubs.py` emits a
dlsym-backed dispatch table, and nothing is on the link line:

> So there is no `-liceoryx2_ffi_c` at link time, and no iceoryx2 headers at build time. The
> harness builds on a machine that has never seen iceoryx2, and it fails at start rather than
> at link if the library is absent.

`sigs/libgodot.sigs` does the same for Godot. The edge builds on a machine with no Godot, and
an image without `libgodot.so` fails at start with a named symbol rather than failing to link.

That answers a question that looked like a design choice and was not. Holding an editor
process warm and shelling out per call are both subprocess management, and neither is how this
project reaches a native library. **The engine is dlopened, in process, like everything else.**

## What a tool is

A name, a JSON schema, and a function. The MCP surface is generic on purpose: the Godot addon
at `v-sekai-multiplayer-fabric/vsekai-godot-mcp` exposes `create_node`, `call_method` and
`call_singleton` and needed no `make_chair` command to build furniture. Neither does this.

Planned first tools, all of which exist today as scripts under `/opt` and want a home:

| tool | what it does | where the code is now |
| --- | --- | --- |
| `csg_build` | CSG solids to glTF, via Godot | `/opt/weft-props/build_props.gd` |
| `gltf_to_usd` | glTF to USD, in metres, verified | `/opt/weft-props/gltf_to_usd.py` |
| `bvh_to_usd` | BVH to USD, unit chosen by measurement | `/opt/weft-motion/bvh_to_usd.py` |
| `motion_gen` | text and constraints to SOMA motion | `/opt/weft-motion/gen_gaps.sh` |
| `render_check` | a contact sheet, for looking at the result | `/opt/weft-motion/render_motion.py` |

Every one of those already emits USD, which is the intermediate this project uses everywhere.
The edge does not introduce a format.

## State

**Not built.** This holds the decision and the signature list. The harness subtree, the
dispatch table, and the tool loop come next. Nothing here runs yet, and the scripts in the
table above do run, which is why they are named.
