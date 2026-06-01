# TwinCAT 3 OOP Introduction — Conveyor Sorting Machine

Source code for the two-part video tutorial series *OOP Introduction* by Ben Harrison (Beckhoff Australia / Coding Bytes). Two complete TwinCAT 3 PLC projects implement identical conveyor sorting machine logic — first in procedural Structured Text, then refactored step by step into full object-oriented code with interfaces and dependency injection.

## Projects

| Project | Folder | Style |
|---|---|---|
| `PLC_NoOOP` | `OOP/OOP/PLC_NoOOP` | Procedural — raw BOOL variables in GVL, all logic flat in MAIN |
| `PLC` | `OOP/OOP/PLC` | OOP — classes, interfaces, dependency injection, two-line MAIN |

Open `OOP/OOP.sln` in TwinCAT XAE to load both projects. Both live inside the same TwinCAT solution.

**TwinCAT version:** 3.1.4024.66+

## The Machine

A conveyor sorting machine with a start/stop control panel and two sensors:

```
[E-Stop (NC)] [Stop (NC)] [Start (NO)]    <- Control panel

         ┌─────────────────────────────┐
[Product]──► [Product Detected (NC)]   │   Conveyor Belt   ──► [Next Conveyor]
         │                             │                         [Full Sensor (NC)]
         └──────────[Motor]────────────┘
```

**Behaviour:** When in run mode, product on the belt triggers the motor for at least 5 seconds (off-delay timer). The motor stops if the downstream conveyor is full. Stop or E-Stop kill run mode immediately.

## Documentation

| Document | Contents |
|---|---|
| [Code Reference](doc/code-reference.md) | Architecture, Mermaid class diagrams, line-by-line walkthrough of both projects |
| [Training: Create the Procedural Program](doc/training-create-procedural-program.md) | Step-by-step guide to building `PLC_NoOOP` from scratch — the starting point for the OOP training |
| [Training: Procedural to OOP](doc/training-procedural-to-oop.md) | Theory, paradigm shift, step-by-step refactoring guide based on the video transcripts |

## Credits

Original tutorial videos by **Ben Harrison** — Beckhoff Australia / Coding Bytes

- [OOP Introduction — Part 1](https://beckhoff-au.teachable.com/courses/1204788/lectures/30170060)
- [OOP Introduction — Part 2](https://beckhoff-au.teachable.com/courses/1204788/lectures/30173203)

Transcripts of both videos are in the `Transcript/` folder.
