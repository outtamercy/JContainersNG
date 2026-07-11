# JContainers NG

A clean-room, standalone reimplementation of **JContainers** for the **CommonLibSSE-NG** era.

Built for Skyrim Anniversary Edition (1.6.x) and VR. No LuaJIT. No Jansson. No Boost. Just `nlohmann/json`, SKSE serialization, and a lot of caffeine.

---

## Why This Exists

The original JContainers is engine infrastructure for Skyrim modding — thousands of mods depend on it. But there was no NG-native build. Rather than hack around the old codebase, this is a ground-up rewrite that:

- Targets **CommonLibSSE-NG** (Monitor221hz template)
- Uses **nlohmann/json** (single header, no external dependencies)
- Implements the **full Papyrus API surface** from the original
- Survives **save/load cycles** via SKSE co-save serialization
- Handles **load-order shifts** correctly via Form string encoding

---

## What's Implemented

| API | Status |
|-----|--------|
| `JValue` (lifetime, I/O, path solving) | ✅ Full |
| `JMap` | ✅ Full |
| `JArray` | ✅ Full (bulk ops, sort, unique, reverse) |
| `JDB` | ✅ Full + SKSE serialization |
| `JFormDB` | ✅ Full |
| `JFormMap` | ✅ Full |
| `JIntMap` | ✅ Full |
| `JString` | ✅ Full |
| `JContainers` utility | ✅ Full |
| `JAtomic` | ❌ Skipped (rarely used) |
| `JLua` | ❌ Skipped (no LuaJIT dependency) |

### Key Architecture Decisions

- **`__jc_ref` live handles**: Nested objects are stored as reference tokens (`{"__jc_ref": 123}`) pointing into a shared registry. This means `JMap.getObj`, `JArray.getObj`, and `JDB.solveObj` return **live references** — mutations propagate instantly.
- **Form serialization**: Forms are encoded as `__formData|PluginName|LocalFormID` strings, making JSON structures immune to load-order reshuffles.
- **Thread-safe `ObjectManager`**: Shared-pointer registry with ref counting. `JValue.retain`/`release` are wired.

---

## Build Requirements

None - it's built.



Outputs are 'JContainersVR.dll' and `JContainers64.dll` in `SKSE/Plugins/`.

---

## Testing

The fastest way to verify:

```papyrus
; In-game console or script
int root = JDB.root()
Debug.Notification("JDB root: " + root)

JDB.solveIntSetter(".test.value", 42, true)
int val = JDB.solveInt(".test.value")
Debug.Notification("Read back: " + val)
```

Save the game, reload, check that `JDB.solveInt(".test.value")` still returns `42`. If it does, serialization is working.

---

## What's Missing / Stubbed
- `JAtomic` — not implemented; rarely used in the wild
- `JLua` — intentionally excluded to avoid LuaJIT dependency

---

## Credits & Attribution

This is a **clean-room reimplementation**. No original JContainers source code was used. The Papyrus API signatures and behavior were derived from the original `.psc` script files and public documentation.

- **Original JContainers**: [ryobg / JContainers](https://github.com/ryobg/JContainers) — the foundation that the entire ecosystem depends on.
- **CommonLibSSE-NG**: [CharmedBaryon](https://github.com/CharmedBaryon/CommonLibSSE-NG) and [Monitor221hz](https://github.com/Monitor221hz/CommonLibSSE-NG-Template-Plugin) — the NG build system that makes this possible.

---

## License

This project follows the same spirit as the original: free to use. If you break your save with it, that's on you.

---

## Status

**Functional.** Builds clean. Survives save/load. Ready for integration testing with mods that depend on JContainers.

Not yet published on Nexus pending author contact (messages blocked, public comment left).
