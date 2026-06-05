# TwinCAT Coding Convention (C#-Style Hybrid Architecture)
This document defines a **strict, deterministic naming and architecture standard** for TwinCAT PLC development.

It combines IEC 61131-3 execution models with a **C#-inspired domain-driven object model**.

Compiler warnings (e.g. 410, 5410) may be disabled, but these rules are **mandatory by convention**.

---

# 1. Core Architecture Model
The system is divided into three explicit layers:

## 1.1 Execution Layer (Active)

```text
FB_*

```
*Cyclic PLC execution*

 Task-driven behavior

*System orchestration*

 Owns time and control flow

---

## 1.2 Domain Layer (Passive Objects)

```text
Valve
RfidReader
Statemachine

```
*No *`FB_`* prefix*

 Pure domain objects

*No cyclic execution*

 No lifecycle management

*Fully passive (react only when called)*

---

## *1.3 Wiring Layer*
`VAR`, `VAR_GLOBAL`, `VAR_INST`

*Dependency Injection model*

 Static compile-time composition only

---

# 2. General Naming Principles
*Names must be ****unambiguous, descriptive, and intention-revealing***

 No abbreviations unless universally understood

*Names must reflect ****domain meaning, not implementation***

---

# *3. Type Naming System*
| Type | Naming Rule | Example |
| --- | --- | --- |
| Interface | I_ + PascalCase | I_Valve |
| Struct | ST_ + PascalCase | ST_ValveState |
| Enum | E_ + PascalCase | E_ValveMode |
| Function | F_ + PascalCase | F_CalcFlow |
| Function Block (PLC) | FB_ + PascalCase | FB_ValveController |
| Class (Domain) | PascalCase (no prefix) | Valve |

---

# *4. Function Block Rule Split*

## *4.1 Standard PLC Function Blocks*

```text
FB_Valve

```
 Cyclic execution

*System-level logic*

 May contain class instances

---

## 4.2 Class Function Blocks (Domain Objects)

```text
Valve

```
*No *`FB_`* prefix*

 Pure object model

*No execution ownership*

 No cyclic logic

---

# 5. Class Function Block Rules

## 5.1 Definition Header (mandatory)

```iecst
{attribute 'no_explicit_call' := '<ClassName> is a class, do not call this POU directly, use a method'}
{attribute 'hide_all_locals'}
FUNCTION_BLOCK ClassName IMPLEMENTS I_Interface

```

---

## 5.2 No VAR_INPUT / VAR_OUTPUT / VAR_IN_OUT
Strictly forbidden.

*Classes do not use call-based data flow*

 All interaction happens via methods and properties

---

## 5.3 Body Must Be Empty
*No logic in FB body*

 All behavior must be in methods

---

## 5.4 No Empty VAR Blocks

```iecst
VAR
END_VAR // ❌ forbidden

```
Must be removed.

---

## 5.5 Interface Requirement
Every class must implement at least one interface:

```iecst
FUNCTION_BLOCK Valve IMPLEMENTS I_Valve

```
If no interface exists → it must be defined first.

---

## 5.6 Execution Rule (Passive Model)
Classes:

*do NOT execute cyclically*

 do NOT contain autonomous behavior

*do NOT schedule themselves*

*They only react when called by *`FB_`* components.*

---

## *5.7 Lifecycle Rule*
*Classes:*

 have NO lifecycle methods

*must NOT contain:*

init()

*start()*

update()

*reset()*

dispose()

Lifecycle is owned by `FB_`.

---

## 5.8 Instantiation Rule (Static Only)
All instances must be declared at compile time:

```iecst
VAR
    _valve : Valve;
END_VAR

```
or:

```iecst
VAR_GLOBAL
    ValveMain : Valve;
END_VAR

```

---

# 6. Interfaces

## Naming

```text
I_Valve
I_Statemachine

```
 PascalCase

---

## Purpose
Interfaces define:

*full public contract*

 compile-time enforcement of behavior

---

# 7. Structs, Enums, Functions

## Structs

```text
ST_ValveState

```

## Enums

```text
E_ValveMode

```

## Functions

```text
F_CalculateFlow

```

---

# 8. Member Variable Rules

## 8.1 General Rule
All member variables start with `_`.

---

## 8.2 Domain Objects

```pascal
_Valve : Valve;
_Statemachine : Statemachine;

```

---

## 8.3 Primitives / Utilities

```pascal
_requestStart : BOOL;
_homingTimer  : TON;
_startRT      : R_TRIG;

```

---

## 8.4 Visibility Encoding
| Prefix | Meaning |
| --- | --- |
| _ | non-public (private/protected concept) |

---

# 9. Methods

## Naming

```text
camelCase

```
Example:

```pascal
requestStart
clearCommands
registerWithParent

```

---

## Visibility Rule
| Scope | Naming |
| --- | --- |
| Public | camelCase |
| Private | _camelCase |
| Protected | _camelCase |

---

# 10. Properties

## Naming

```text
camelCase

```

---

## Visibility Rule
| Scope | Naming |
| --- | --- |
| Public | camelCase |
| Private | _camelCase |
| Protected | _camelCase |

---

## Boolean Properties
Preferred:

```text
isActive
hasError
canWork

```
Allowed:

*logical state (*`isEnabled`*, *`isInterlocked`*)*

*Not allowed:*

 physical interpretation (`isHigh`, `isOpen` if purely wiring-based)

---

# 11. Hardware Signals

## Prefixes
| Prefix | Meaning |
| --- | --- |
| DI_ | digital input |
| DO_ | digital output |
| AI_ | analog input |
| AO_ | analog output |

---

## Rules
*Never use *`_`* prefix*

 Must describe **physical signal meaning**

*Always camelCase after prefix*

*Example:*

```pascal
DO_valveOpen : BOOL;
DI_emergencyStop : BOOL;

```

---

# *12. HMI Structs*
*Always named:*

```text
hmi

```
*Example:*

```pascal
{attribute 'TcHmiSymbol.AddSymbol'}
hmi : ST_Valve_HMI;

```

---

# *13. Null Objects*
*Always hidden:*

```pascal
{attribute 'hide'}
_Null_WorkSolenoid : DigitalOutput;

```

---

# *14. Constants*

```text
UPPER_CASE_WITH_UNDERSCORES

```
*Example:*

```pascal
MAX_RETRIES     : DINT := 3;
DEFAULT_TIMEOUT : TIME := T#10S;

```

---

# *15. VAR_GENERIC CONSTANT*

```iecst
VAR_GENERIC CONSTANT
    NumberOfFeedbacks : UDINT := 1;
END_VAR

```

---

# *16. Clean Code Principles*

## *16.1 Intention-Revealing Names*
*Prefer:*

```pascal
_elapsedTimeSinceLastCycle

```

---

## *16.2 One Word per Concept*
*Choose one:*

 get

*fetch*

 retrieve

Do not mix.

---

## 16.3 Avoid Disinformation
Avoid generic terms unless domain-valid:

*Manager*

 Controller

*Handler*

*Only use if it reflects actual responsibility.*

---

## *16.4 Meaningful Context*
*Prefer:*

```pascal
valveState
homingState

```
*Not:*

```pascal
state

```

---

## *16.5 Avoid Mental Mapping*
*Avoid single-character names except loops:*

```pascal
i, j

```

---

# *17. Folder Organization (TwinCAT XAE)*
*Recommended structure:*

```plaintext
FunctionBlock/
  publicMethod
  Protected/
    _stateMachine
  Private/
    _clearCommands
  I_Interface/
    Commands/
    Status/

```

---

# *18. Unit Testing Convention*

## *Class naming*

```text
C##_ClassName_TEST

```
*Example:*

```text
C07_SimpleModeModule_TEST

```

---

## *Test method naming*

```text
T##_Description_ExpectedResult

```
*Example:*

```text
T01_InitialState_IsAutoStop

```

---

## *Variable placement*

```pascal
VAR_INST  // expected + actual
VAR       // SUT

```

---

# *19. Final Architectural Summary*

## *Execution Layer*
`FB_` active runtime logic

*owns system timing*

## *Domain Layer*
 class FBs (`Valve`)

*passive objects*

 no execution responsibility

## Wiring Layer
*static instantiation*

 dependency injection preferred

*optional global instances allowed*

---

# *20. Core Design Principles*
 Execution is always owned by `FB_`

*Domain objects are always passive*

 No runtime instantiation

*No hidden behavior*

 Everything is explicit, deterministic, and statically defined