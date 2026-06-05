# Exercise 10 — Summary, Code Review, and Outlook

## What This Training Covered

This series walked from a single IEC 61131-3 function block all the way to a framework with interfaces, inheritance, dependency injection, design patterns, and a self-registering HMI forceables layer. The exercises were deliberately sequenced so each new concept solved a problem left open by the previous one.

---

## Exercise Recap and Design Decisions

### Exercise 01 — Classes, Encapsulation, Abstraction

**What was built:** `I_DigitalInput`, `DigitalInputNO`, `DigitalInputNC`, `DigitalInputDummy`

**Key decisions:**
- Hardware variables declared private; exposed only via `isActive` / `isNotActive` — not `isHigh` / `isLow`. Property names reveal application intent, not hardware detail.
- `DigitalInputDummy` returns `TRUE` for both properties intentionally — the **Null Object** pattern: a permissive placeholder that never blocks code flow. Logical inconsistency is by design.
- `{attribute 'no_explicit_call'}` enforces the class model at compile time. The error message is the documentation.

**Patterns:** Null Object, Interface as Contract

---

### Exercise 02 — Base Class, Inheritance, Constructor

**What was built:** Abstract `Base` with `_name`, `_path`, `trace`, `FB_Init`

**Key decisions:**
- `ABSTRACT` prevents direct instantiation. The only valid use is inheritance.
- `{attribute 'instance-path'}` + `{attribute 'noinit'}` populate `_path` automatically — the runtime knows where every object lives. No programmer overhead after the initial declaration.
- `FB_Init` is TwinCAT's constructor. Passing name at instantiation (`DigitalInputNO('I201.1')`) makes the declaration the source of truth for object identity.
- `trace` delegates to `ADSLOGSTR` directly in this exercise — the hard-coded dependency that exercise 03 removes.

**Patterns:** Template Method (trace with fallback), Abstract Base Class

---

### Exercise 03 — Dependency Injection, the Logger

**What was built:** `I_Logger`, `LoggerADS`, `GVL_Logger`, logger wired into `Base.FB_Init`

**Key decisions:**
- `Base` no longer calls `ADSLOGSTR` directly. It calls `_logger.trace(message)`. The logging destination is a dependency supplied from outside — Dependency Inversion Principle (SOLID D).
- `GVL_Logger.logger` is the global `LoggerADS` instance. Passing it via `FB_Init` preserves the DI contract: `Base` knows only `I_Logger`, not `LoggerADS`, not `GVL_Logger`.
- `__ISVALIDREF` guard in `trace` makes the logger optional — ADSLOGSTR fallback ensures safety.
- Global singleton logger was explicitly considered and rejected for the general case. The GVL approach is a pragmatic middle ground: `Base` is decoupled; callers don't need verbose wiring chains.

**Patterns:** Dependency Injection, Strategy (logger as interchangeable algorithm), Null Object (ADSLOGSTR fallback)

**SOLID:** D (Dependency Inversion), O (Open/Closed — new loggers don't change Base)

---

### Exercise 03a — Severity Levels (Described)

**Key decisions:**
- `TcEventSeverity` from `Tc3_EventLogger` as the standard vocabulary — avoids reinventing severity enums.
- Default severity in a **parameter list** (non-constant GVL), not a constant. Constants are baked into the binary; parameter list values can be overridden per deployment without recompiling. Different machines, different log verbosity, same binary.
- `IF severity <> TcEventSeverity.Verbose` as the fallback sentinel — a known trade-off documented in the exercise.

**SOLID:** S (SRP — a class should have one reason to change; configuration is separated from code)

---

### Exercise 03b — Observer Pattern, Multi-Destination Logging (Described)

**Key decisions:**
- The Observer pattern solves fan-out: one `Logger` subject, many `I_LogObserver` subscribers.
- `LoggerADS` and `LoggerMQTT` become observers — neither knows about the other.
- `LoggerMQTT` uses `FB_IotMqttClient` (TF6701) which must be called cyclically — infrastructure FBs are not domain classes and do not carry `{attribute 'no_explicit_call'}`.
- A ring buffer in `LoggerMQTT` decouples synchronous notification from asynchronous MQTT delivery.
- HiveMQ public broker used for training. Topic uses a random UUID segment — anonymization on shared infrastructure.

**Patterns:** Observer / Publish-Subscribe, Strategy (concrete logger as strategy), Ring Buffer
**SOLID:** O (add new loggers without changing Logger or LoggerADS)

---

### Exercise 04 — Proxy and Decorator, the Forceable Signal

**What was built:** `I_Bool`, `I_ForceableBOOL`, `BoolSignal`, `BoolForceable`

**Key decisions:**
- `BoolForceable` extends `BoolSignal` — inheritance-based Proxy. The real subject (`BoolSignal`) is an implementation detail; external code holds `I_Bool` or `I_ForceableBOOL`.
- `SUPER^.value` in the non-forced path makes the proxy completely transparent.
- `VAR_IN_OUT` in `bind` — the only way to capture a `REFERENCE TO` from an external variable in TwinCAT. `ADR` was the alternative; `VAR_IN_OUT` + `REF=` is cleaner.
- Decorator was explicitly discussed as the alternative at the `I_DigitalInput` level. Proxy at signal level was chosen because NC/NO inversion still applies to the forced value.

**Patterns:** Proxy, discussed Decorator
**SOLID:** O (new forcing behavior without changing DigitalInputNC's interface)

---

### Exercise 04a — YAGNI, Removing BoolSignal (Described)

**Key decision:** `BoolSignal` exists in exercise 04 to show the Proxy pattern's structural participants. In production code it should be removed and its members folded into `BoolForceable` — YAGNI. Reintroduce it only when a second concrete non-forceable `I_Bool` use case appears.

---

### Exercise 05 — Registry, Self-Registration

**What was built:** `Forceables`, `GVL_Forceables`, `ST_ForceAble`, auto-registration in `BoolForceable.FB_Init`

**Key decisions:**
- Registry (Fowler, PEAA) provides a central, well-known access point to all `BoolForceable` instances.
- Two-phase initialization: inline VAR declaration gives a safe-default name and logger; the explicit `FB_Init` call refines both. The `name <> ''` guard prevents phantom registrations.
- `ST_ForceAble` is the HMI bridge — a plain struct the HMI can read and write. The registry holds interface references; the HMI sees structs. The two are synchronised by `renew()` every scan.
- The registry is deliberately global — unlike the logger, it cannot be injected without forwarding it through every device constructor. The requirement (one HMI sees all forceables) justifies the global.

**Patterns:** Registry (Fowler), Self-Registration idiom
**SOLID:** S (Forceables has one job: manage the list)

---

## Patterns and Principles — Quick Reference

| Pattern | Where used | Why |
|---|---|---|
| Null Object | `DigitalInputDummy` | Permissive placeholder — never blocks code |
| Template Method | `Base.trace` | Fallback to ADSLOGSTR when no logger |
| Strategy | `I_Logger` / `LoggerADS` | Swap logging destination without changing Base |
| Dependency Injection | `Base.FB_Init(logger)` | Caller decides the logger; Base is decoupled |
| Observer | `Logger` subject / `I_LogObserver` | Fan out one message to many destinations |
| Proxy | `BoolForceable` / `BoolSignal` | Intercept signal reads for force override |
| Decorator | Discussed as alternative | Add behavior to `I_DigitalInput` from outside |
| Registry | `Forceables` / `GVL_Forceables` | Central HMI-accessible list of all forceables |
| Self-Registration | `BoolForceable.FB_Init` | Object announces itself at construction time |

| SOLID | Where applied |
|---|---|
| S — Single Responsibility | Each class has one job: `Base` = identity+trace, `BoolForceable` = force proxy, `Forceables` = registry |
| O — Open/Closed | New loggers extend the system; `Logger` and `Base` are closed for modification |
| L — Liskov Substitution | `DigitalInputNO`, `DigitalInputNC`, `DigitalInputDummy` are all substitutable as `I_DigitalInput` |
| I — Interface Segregation | `I_Bool` / `I_ForceableBOOL` split read-only from force-capable; consumers hold the narrowest interface they need |
| D — Dependency Inversion | `Base` depends on `I_Logger`, not `LoggerADS`; `Forceables` depends on `I_ForceableBOOL`, not `BoolForceable` |

---

## Deep Dive — Current Code Review

The framework as built teaches the right ideas. In production it has several gaps worth addressing before shipping.

### Issue 1 — `DigitalInputNO` and `DigitalInputDummy` are not forceable

`DigitalInputNC` uses `BoolForceable` and registers with the HMI. `DigitalInputNO` and `DigitalInputDummy` still read `_input` directly. This means roughly two-thirds of the device classes are invisible to the Forceables HMI panel.

**Better:** apply the same `BoolForceable` pattern to all three classes. The pattern in `DigitalInputNC.FB_Init` is the template — one declaration line change and two body lines per class.

---

### Issue 2 — `BoolSignal` still exists

Exercise 04a explicitly describes removing `BoolSignal` and folding its members into `BoolForceable`. The source files still contain both. `BoolSignal` is an intermediate abstraction with no standalone use case yet.

**Suggestion:** If no non-forceable `I_Bool` consumer exists, remove `BoolSignal` and apply the exercise 04a refactor. If the team anticipates safety inputs that must never be forceable, keep `BoolSignal` — it IS the right abstraction then.

---

### Issue 3 — `LoggerADS` lacks class pragmas

`LoggerADS` was created without `{attribute 'no_explicit_call'}` and `{attribute 'hide_all_locals'}`. It can be accidentally called as a function block and its internals are visible in the watch window.

**Better:** add both pragmas. It is a class in the framework sense.

---

### Issue 4 — Naming inconsistency: `logger` vs `Logger`

`DevicesExample` passes `GVL_Logger.logger` (lowercase). `DigitalInputNC` inline init references `GVL_Logger.Logger` (uppercase). TwinCAT identifiers are case-insensitive at runtime but the inconsistency reads as a typo and will confuse anyone reading the code.

**Better:** pick one casing and apply it everywhere. The coding style guide uses PascalCase for properties and methods — `Logger` is the property name; `GVL_Logger.Logger` is consistent with that. Update `DevicesExample` to match.

---

### Issue 5 — `I_ForceableBOOL` naming is inconsistent with the framework convention

The framework uses PascalCase throughout: `I_DigitalInput`, `I_Logger`, `I_Bool`. The interface `I_ForceableBOOL` breaks this with all-caps `BOOL`.

**Better:** rename to `I_ForceableBool` for consistency.

---

### Issue 6 — `BoolForceable` has a hard dependency on `GVL_Forceables`

`BoolForceable.FB_Init` calls `GVL_Forceables.manager.register(THIS^)` directly. This is the global dependency the registry justifies — but it means `BoolForceable` cannot be used in a different project without `GVL_Forceables` being present.

**Practical suggestion:** consider making the registry optional via a compile-time `{IF defined(...)}` block, or provide a `BoolForceableSimple` variant without self-registration for projects that do not use the HMI forceables panel.

---

### Issue 7 — Exercises 03a, 03b, 04a are described but not implemented

The severity level system, the Observer logger with MQTT, and the `BoolSignal` removal are all documented but the code reflects the earlier versions. Students reading the exercises expect the code to match.

**Suggestion:** mark these clearly in the framework overview as "described, not yet implemented" with a status column, or implement them in a separate `PLC_OOP` project that can serve as the reference implementation.

---

### Issue 8 — No output (actuator) classes

The framework covers digital inputs thoroughly. There are no `DigitalOutput`, `I_Actuator`, or actuator base classes. A real machine has at least as many outputs as inputs.

---

## What to Add Next — Roadmap

### Near term — complete what was started

| Task | What | Why |
|---|---|---|
| Fix Issue 1 | Apply `BoolForceable` to `DigitalInputNO` | HMI forceables panel covers all inputs |
| Fix Issue 3 | Add pragmas to `LoggerADS` | Consistent class model |
| Fix Issue 4 | Unify `Logger` casing | Code quality |
| Exercise 04a | Remove `BoolSignal` | YAGNI — one class is better than two |

### Medium term — fill the gaps

**Add `I_DigitalOutput` and concrete classes.**
Mirror the digital input pattern: `I_DigitalOutput` with `activate()`, `deactivate()`, `isActive`, `isNotActive`. `DigitalOutputNO`, `DigitalOutputNC`. The same `BoolForceable` proxy applies to the output signal — the HMI can force outputs too, which is often more important than forcing inputs during commissioning.

**Implement the Observer logger (exercise 03b).**
`Logger` subject, `I_LogObserver`, `LoggerADS` as observer, `LoggerMQTT` stub. Even without a live broker, the structural refactor shows the pattern in working code.

**Add severity levels to the logger (exercise 03a).**
`GVL_Parameters.DefaultTraceSeverity`, severity parameter on `I_Logger.trace`, mapping in `LoggerADS`.

### Long term — framework to library

**Extract `Base` into a standalone TwinCAT library.**
A company library gives every project the same base class without copy-paste. Library development in TwinCAT requires versioning discipline — plan for breaking changes early.

**Add `I_Module` and a module base class.**
A module is a grouping of devices that manages its own lifecycle: `init()`, `run()`, `fault()`, `reset()`. `Base` provides identity and logging; the module base adds the lifecycle contract. The conveyor sorting machine from the procedural training becomes a module that owns its sensors and actuators.

**Add a lifecycle interface.**
```
I_Lifecycle
  init() → run setup sequences
  cyclic() → called every scan
  fault() → entered on detected error
  reset() → attempt recovery
```
This is a form of the **State** pattern embedded in the interface contract.

**Add a factory for device creation.**
Instead of:
```iecst
door : DigitalInputNC('Door sensor', GVL_Logger.logger);
```
A `DeviceFactory.createNC(name)` returns `I_DigitalInput` and handles logger injection internally. Decouples the caller from the concrete type entirely.

**Consider TwinCAT namespaces.**
When packaging as a library, use TwinCAT's namespace feature to prevent name collisions with customer code.

---

## Practical Advice for Production Use

**Not everything needs to be a class.** The framework's class rules (no explicit call, no VAR_INPUT in FB body, no empty VAR blocks) are right for domain objects. Infrastructure FBs — state machines, timers, communication handlers — are better left as callable function blocks. The `Forceables` manager is already an example of this pragmatic boundary.

**Interface calls have overhead.** Every call through an interface reference in TwinCAT involves a vtable lookup. In a 250 μs cycle with 500 devices, this can matter. Benchmark before adding interface layers to tight inner loops. The outer structure (devices, modules, machines) is the right place for interfaces; the inner physics (signal conditioning, ramp generation) may be better served by direct calls.

**Online change compatibility.** Interface references stored in GVLs survive online changes. Interface references stored in PROGRAM variables may be invalidated. The `BoolForceable` registration that happens in `FB_Init` is called once at startup — after an online change that reinitializes the program, the registration happens again. This is correct behavior, but it means the `Forceables` count may jump if you watch it during development.

**The `_path` attribute is read-only and runtime-only.** It is populated by the TwinCAT runtime, not by the programmer. It is not available at download time or in offline simulation. Code that depends on `path` for anything other than diagnostics should be reviewed.

**A parameter list is not a backup.** `GVL_Parameters.DefaultTraceSeverity` is configurable per deployment — but it is not a persistent user setting. If the operator changes verbosity at runtime and the controller restarts, the GVL resets to its compiled default. If you need user-persistent settings, look at `RETAIN` variables or a configuration file approach via `FB_FileOpen`.
