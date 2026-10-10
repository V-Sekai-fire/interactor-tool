# interactor-tool

Design for the tool plane: MCP frames over iceoryx2 shared memory, so a new capability is a registered tool rather than a new container.

## What it is for

MCP is JSON-RPC framing, so the plane is designed to carry its frames on the shared-memory bus with no socket to defend, and a tool call that carries a mesh, a motion or a stage is written once and read in place. A tool has no tick and holds no state between calls, so one plane holds a tool table and adding a capability is adding a row. A tool is a name, a JSON schema and a function; `TOOLS.md` describes the shape. Godot's C ABI is listed in a signature file to be reached by dlopen, so the plane is designed to build without the engine and to fail at start with a named symbol when the library is absent.

## Build

There is nothing to build yet: the repository holds the design and the signature list, and no plane code.

## Licence

MIT. See [LICENSE](LICENSE).
