# fabric-tool-plane

One plane that runs tools, so a new capability is a registered tool and not a new container.

## Why this is a plane

The first draft of this repository called itself an edge, on the reasoning that MCP is a wire
protocol and `CLAUDE.md` says:

> A plane has no networking. An edge is a plane with networking.

That was wrong about MCP rather than about the rule. **MCP is JSON-RPC framing.** The
transport is incidental to it, and the usual ones, stdio and HTTP, are conventions rather than
part of the protocol. Carry the same frames over iceoryx2 and there is no networking left to
argue about, so this is a plane and takes no exception.

It is also the better design on its own terms, which is the part that matters more than the
naming:

- **Zero copy is worth more here than almost anywhere.** A tool call carries a mesh, a motion,
  or a stage. Over HTTP that is a serialise, a copy, and a parse. Over iceoryx2 the caller
  writes it once and the tool reads the same bytes.
- **No listening socket.** An edge has to be defended. A plane on the bus does not have the
  surface to defend.
- **It composes.** Every other plane already reaches the bus. A tool becomes callable from the
  crowd plane the same way the store is, and nothing has to learn a second way to talk.

## Why one plane and not one per tool

Each capability wanted its own repository, its own container, and its own Fly app: one to
build props, one to convert motion, one to generate. Each is a deploy, an image push, and a
cold start, for a job that runs for seconds.

A tool is not a plane in its own right. It has no tick, it holds no state between calls, and
it does not need a core to itself. So the shape that fits is one plane with a tool table, and
adding a capability is adding a row.

## How a tool reaches a library

The same way the harness reaches iceoryx2, because weft made this decision once already.
`fabric-harness/iceoryx2.sigs` lists the C ABI it calls, `generate_stubs.py` emits a
dlsym-backed dispatch table, and nothing is on the link line:

> So there is no `-liceoryx2_ffi_c` at link time, and no iceoryx2 headers at build time. The
> harness builds on a machine that has never seen iceoryx2, and it fails at start rather than
> at link if the library is absent.

`sigs/libgodot.sigs` does the same for Godot. The plane builds on a machine with no Godot, and
an image without `libgodot.so` fails at start with a named symbol rather than failing to link.

That also answers a question that looked like a design choice and was not. Holding an editor
process warm and shelling out per call are both subprocess management, and neither is how this
project reaches a native library. **The engine is dlopened, in process, like everything else.**

## What a tool is

A name, a JSON schema, and a function. See `TOOLS.md`. The surface is generic on purpose: the
Godot addon at `v-sekai-multiplayer-fabric/vsekai-godot-mcp` exposes `create_node`,
`call_method` and `call_singleton`, and that was enough to build five pieces of furniture
without one furniture-specific command being designed.

| tool | what it does | where the code is now |
| --- | --- | --- |
| `csg_build` | CSG solids to glTF, via Godot | `/opt/weft-props/build_props.gd` |
| `gltf_to_usd` | glTF to USD, in metres, verified | `/opt/weft-props/gltf_to_usd.py` |
| `bvh_to_usd` | BVH to USD, unit chosen by measurement | `/opt/weft-motion/bvh_to_usd.py` |
| `motion_gen` | text and constraints to SOMA motion | `/opt/weft-motion/gen_gaps.sh` |
| `render_check` | a contact sheet, for looking at the result | `/opt/weft-motion/render_motion.py` |

Every one of those already emits USD, which is the intermediate this project uses everywhere.
The plane introduces no format.

## State

**Not built.** This holds the decision and the signature list. The harness subtree, the
dispatch table, the JSON-RPC framing over iceoryx2, and the tool loop come next. The scripts
in the table above do run today, which is why they are named rather than described.
