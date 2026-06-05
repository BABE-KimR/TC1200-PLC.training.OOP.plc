# Framework OOP Training Series — Overview

This series introduces the idea of building a **company-owned OOP framework** in TwinCAT 3 Structured Text. Rather than delivering a finished library, each exercise adds one concept at a time so the reader can follow the thinking behind every design decision.

The exercises live in the `PLC_FrameworkOOP` TwinCAT project. They are self-contained: no machine is wired up — the focus is purely on the framework structure and the OOP concepts it demonstrates.

---

## Why a Company Framework?

A well-designed base-class library gives every project in your portfolio:

- A **common vocabulary** — all developers use the same terms and interfaces
- **Consistent behaviour** — device handling, error reporting, and lifecycle management work the same way everywhere
- **Faster onboarding** — new developers learn the pattern once and apply it to every machine

The framework is not meant to be imported as a black box. It is meant to be *understood*, adapted, and owned by the team.

---

## OOP Concepts Covered

The exercises build on each other in this order:

| Concept | What it means in PLC context |
|---|---|
| **Encapsulation** | Hiding raw hardware variables inside a function block; external code only sees methods and properties |
| **Abstraction** | Defining interfaces that describe *what* an object does, not *how* |
| **Inheritance** | Sharing common behaviour through a base function block that concrete classes extend |
| **Polymorphism** | Writing code against an interface so the same call works for any concrete type behind it |
| **Design patterns** | Proven solutions (State, Observer, Factory, …) adapted to the PLC scan-cycle model |

---

## Framework Rules

One reference document applies to every exercise. Read it before starting and keep it open while coding.

→ [TwinCAT Coding Style](TwinCAT-coding-style.md) — architecture model, type system, class rules, naming conventions, pragmas, and folder structure for all IEC 61131-3 elements in this framework

---

## Exercises

Each exercise adds one layer to the framework. Read the guide before opening TwinCAT — the design explanations are the main learning content, not the code listing.

| # | Title | Concepts introduced | Guide |
|---|---|---|---|
| 01 | Digital Input Classes | OOP class vs IEC FB, encapsulation, abstraction, interface as contract, placeholder pattern | [exercise-01-digital-input.md](exercise-01-digital-input.md) |
| 02 | Base Class and Inheritance | Abstract class, inheritance (`EXTENDS`), constructor (`FB_Init`), instance path (reflection), shared diagnostic infrastructure (`trace`) | [exercise-02-base-class-inheritance.md](exercise-02-base-class-inheritance.md) |
| 03 | Dependency Injection and the Central Logger | Dependency Inversion Principle (SOLID D), dependency injection via constructor, global singleton vs DI, Strategy pattern, null-safe fallback | [exercise-03-dependency-injection.md](exercise-03-dependency-injection.md) |
| 03a | Severity Levels in the Logger *(described, not implemented)* | `TcEventSeverity` from `Tc3_EventLogger`, per-instance severity, parameter list vs constant list (SOLID S), ISP trade-off on interface design | [exercise-03a-severity-levels.md](exercise-03a-severity-levels.md) |
| 03b | Observer Pattern and Multi-Destination Logging *(described, not implemented)* | Observer / Publish-Subscribe (GoF), Subject and Observer interfaces, fixed-size subscriber lists, cyclic vs synchronous notification, TF6701 MQTT | [exercise-03b-observer-pattern.md](exercise-03b-observer-pattern.md) |
| 04 | Proxy and Decorator: the Forceable Signal | Proxy pattern (GoF), Decorator pattern (GoF), Proxy vs Decorator distinction, inheritance-based Proxy, `BoolSignal` / `BoolForceable` hierarchy, `DigitalInputNC` with forcing | [exercise-04-proxy-decorator.md](exercise-04-proxy-decorator.md) |
| 04a | `BoolForceable` Without the Intermediate Class *(described, not implemented)* | YAGNI principle, removing premature abstraction, when to reintroduce `BoolSignal` | [exercise-04a-bool-forceable-simplified.md](exercise-04a-bool-forceable-simplified.md) |
| 05 | Registry and Self-Registration: the Central Forceables List | Registry pattern (Fowler), self-registration idiom, initialization order in TwinCAT, HMI bridge via struct array, Registry vs Singleton trade-off | [exercise-05-registry-self-registration.md](exercise-05-registry-self-registration.md) |
| 06 | Polymorphism: One Loop, Three Types | Interface arrays, single-loop dispatch, Liskov Substitution Principle (SOLID L), virtual dispatch through `I_DigitalInput` | [exercise-06-polymorphism.md](exercise-06-polymorphism.md) |
| 07 | Template Method: Abstract Base Class for Digital Inputs | Abstract function blocks, abstract properties, Template Method pattern (GoF), enforced invariants, overriding concrete template steps | [exercise-07-template-method.md](exercise-07-template-method.md) |
| 10 | Summary, Code Review, and Outlook | Full pattern and SOLID recap, code audit with 8 specific issues, production roadmap, practical advice | [exercise-10-summary-and-outlook.md](exercise-10-summary-and-outlook.md) |
| 10a | Final Exam | 20 questions across concepts, code reading, design, and critical thinking; marking guide for key questions | [exercise-10a-exam.md](exercise-10a-exam.md) |

---

## How to Use This Series

1. Read the exercise guide from top to bottom before touching TwinCAT.
2. Follow the step-by-step instructions to reproduce the code yourself.
3. Check your result against the reference code in `PLC_FrameworkOOP`.
4. The guide explains *why* each design decision was made — treat those explanations as the main learning content, not the code listing itself.

---

## Relation to Other Projects in This Repository

| Project | Style | Purpose |
|---|---|---|
| `PLC_NoOOP` | Procedural | Baseline: shows the problems OOP solves |
| `PLC_OOP` | Object-oriented | First refactor: interfaces and dependency injection applied to one machine |
| `PLC_FrameworkOOP` | Framework OOP | This series: building reusable abstractions a team can share across projects |
