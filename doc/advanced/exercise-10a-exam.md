# Exercise 10a — Final Exam

These questions test understanding of the design decisions, patterns, and principles covered in the training. Some questions have a single correct answer; others are open-ended and reward reasoning over recall. For open-ended questions, a good answer explains *why*, not just *what*.

---

## Part A — Concepts (Short Answer)

**A1.** The `DigitalInputDummy` class returns `TRUE` for both `isActive` and `isNotActive`. A student objects: "That's logically impossible — a real input can't be both active and not-active at the same time." How do you defend the design?

---

**A2.** `Base` is declared `ABSTRACT`. What does this enforce at compile time? Why is this the right choice for a base class that is never meant to be used directly?

---

**A3.** Name the SOLID principle violated by the following code, and explain why it is a violation:

```iecst
FUNCTION_BLOCK DigitalInputNO EXTENDS Base IMPLEMENTS I_DigitalInput
VAR
    _input AT %I* : BOOL;
END_VAR

METHOD trace_to_ads
ADSLOGSTR(msgCtrlMask := ADSLOG_MSGTYPE_HINT, msgFmtStr := _input.name, strArg := '');
```

---

**A4.** The `GVL_Logger.logger` instance is accessible from anywhere in the project. In exercise 03 we argued strongly against using a global singleton for the logger. Here we use one anyway. Is this a contradiction? Justify your answer.

---

**A5.** What is the difference between the Proxy pattern and the Decorator pattern? Give one sentence for each. Then explain which one `BoolForceable` implements and why.

---

**A6.** `BoolForceable.FB_Init` contains this guard:

```iecst
IF name <> '' THEN
    GVL_Forceables.manager.register(THIS^);
END_IF
```

What happens without this guard? Walk through TwinCAT's initialization sequence for `DigitalInputNC` and show the specific failure.

---

**A7.** The `Forceables.register` method scans the entire array on every call. With `MAXFORCEABLES = 100` this is not expensive. If the framework grew to 10 000 forceables, how would you improve the lookup without changing the public interface?

---

**A8.** What does `{attribute 'qualified_only'}` do on `GVL_Forceables`? Why is it a good practice for GVLs in a framework?

---

## Part B — Code Reading

**B1.** Trace through what happens, step by step, when `MAIN` runs for the first time after a cold start. List the initialization calls in order and state what each one does.

---

**B2.** `DevicesExample` declares:

```iecst
ncInput : DigitalInputNC('I201.2', GVL_Logger.logger);
```

`DigitalInputNC.FB_Init` does not contain an explicit call to `SUPER^.FB_Init`. After startup, `ncInput.name` nonetheless returns `'I201.2'` correctly. Explain why — describe TwinCAT's FB_Init call behaviour for derived function blocks.

---

**B3.** Consider this code in `Forceables.renew()`:

```iecst
IF GVL_Forceables.arrForceables[i].forced THEN
    _objects[i].force(GVL_Forceables.arrForceables[i].forceValue);
ELSE
    _objects[i].release();
END_IF
GVL_Forceables.arrForceables[i].value := _objects[i].value;
```

An operator sets `forced := TRUE` and `forceValue := TRUE` in the HMI. Then they change `forceValue` to `FALSE` without clearing `forced`. What does the HMI display for `value` on the next scan? Show your reasoning.

---

**B4.** `BoolForceable.value` is implemented as:

```iecst
IF _isForced THEN
    value := _forced;
ELSE
    value := SUPER^.value;
END_IF
```

If `bind` was never called on the parent `BoolSignal`, what does `SUPER^.value` return? Is this safe? What could go wrong at the device level?

---

**B5.** `DigitalInputNC.isActive` is:

```iecst
isActive := NOT _signal.value;
```

An operator forces `_signal` to `TRUE` via the HMI. What does `isActive` return? What does `isNotActive` return? Is this the expected behaviour for a normally-closed input?

---

## Part C — Design

**C1.** You must add `DigitalInputNO` to the Forceables HMI panel so operators can force normally-open inputs during commissioning. Describe every change needed — which files, which lines, what you add. You do not need to write the full code, but be specific enough that a colleague could implement it without asking questions.

---

**C2.** A new requirement: the HMI panel should show not just the device's signal name but a user-readable description — for example, "Emergency stop — cabinet door" rather than `'I201.1'`. The description is provided at instantiation, not derived from the IO address.

Design the minimal change to the framework to support this. Which interface(s) change? Which classes? Is this a breaking change? How do you handle existing code that passes only one constructor argument?

---

**C3.** A student proposes replacing the `I_Logger` interface with a direct call to `GVL_Logger.logger.trace(message)` inside `Base.trace`. Their argument: "We always use `GVL_Logger.logger` anyway, so the interface is just overhead." Write a two-paragraph response explaining what is wrong with this argument, using specific examples from the exercises.

---

**C4.** You need to add a `DigitalOutputNC` class — a normally-closed digital output that activates by setting the hardware output to `FALSE`. Design the class:
- Which interfaces does it implement?
- Does it extend `Base`? Does it use `BoolForceable`?
- What does `activate()` look like?
- Sketch the VAR block and FB_Init.

---

**C5.** The Observer pattern in exercise 03b says `LoggerMQTT` cannot have `{attribute 'no_explicit_call'}` because it needs to be called cyclically. A student argues: "Then it's not really a class and breaks the framework's design rules." Write a response that distinguishes between domain classes and infrastructure services, and explains why the same rules don't apply to both.

---

## Part D — Critical Thinking

**D1.** The framework uses `REFERENCE TO BOOL` in `BoolSignal` via `VAR_IN_OUT` in the `bind` method. A colleague proposes using `POINTER TO BOOL` and `ADR(_input)` instead. List one advantage and one disadvantage of each approach.

---

**D2.** `MAXFORCEABLES = 100` is declared as `VAR_GLOBAL CONSTANT`. A student suggests changing it to `VAR_GLOBAL` so it can be adjusted at runtime. What is the specific technical consequence of making this change in TwinCAT? Is the student's suggestion valid?

---

**D3.** Examine the following proposed change to `DigitalInputDummy`:

```iecst
FUNCTION_BLOCK DigitalInputDummy EXTENDS Base IMPLEMENTS I_DigitalInput
VAR
    _isActive    : BOOL := FALSE;
    _isNotActive : BOOL := FALSE;
END_VAR
```

Where both properties return the corresponding member variable, and a `setActive(value : BOOL)` method sets them consistently. Compare this to the current implementation. What capability does the new version add? What does it lose? When is the current version preferable?

---

**D4.** The `Forceables.register` method checks `IF _objects[i] = obj THEN` to detect duplicate registrations. The `_objects` array holds `I_ForceableBOOL` interface references. What exactly is being compared when you compare two interface references with `=`? Would two different `BoolForceable` instances that hold the same hardware input register as duplicates?

---

**D5.** After completing this training, a colleague says: "Great, now we should rewrite all our existing procedural programs as OOP classes." Argue for and against this position. What is the deciding factor?

---

## Marking Guide — Selected Questions

**A1.** Full marks for: the Null Object pattern, the permissive placeholder concept, the design trade-off (logical inconsistency is deliberate), and the practical value (code runs without blocking on a hardware decision). Partial marks for "it's for testing" without explaining why always-TRUE is the right choice.

**A3.** The violation is **Dependency Inversion** (SOLID D) — the method calls `ADSLOGSTR` directly (a concrete low-level mechanism) rather than through the `I_Logger` abstraction. Secondary: the method name reveals implementation (`trace_to_ads`) not intent — naming violation. The `_input.name` reference also doesn't exist — `_input` is `AT %I* : BOOL` which has no `.name` — this is also a factual error in the code; good students should spot this.

**B2.** TwinCAT automatically calls the `FB_Init` of every base function block in the inheritance chain before the derived `FB_Init` body runs. For `DigitalInputNC`: `Base.FB_Init` is called first (setting `_name`, `_logger`, `_path`), then `DigitalInputNC.FB_Init` body runs and handles `_signal` initialisation. No explicit `SUPER^.FB_Init` call is needed or correct — adding one would invoke `Base.FB_Init` a second time.

**B3.** The HMI shows `value = FALSE`. The `force(FALSE)` call is made (because `forced = TRUE`), so `_isForced = TRUE` and `_forced = FALSE`. On the value update line, `_objects[i].value` returns `FALSE` (the forced value). The operator set `forceValue` to `FALSE`, and that is exactly what is applied and displayed.

**B4.** `BoolSignal.value` returns `FALSE` (the default for `BOOL` when the `__ISVALIDREF` guard fails). This is safe at the signal level — no crash. At the device level, `DigitalInputNC.isActive` returns `NOT FALSE = TRUE` — the device appears active even though no hardware is connected. This could mask a wiring fault or cause unintended logic to execute.

**D4.** Interface references in TwinCAT are compared by their internal pointer value — the address of the underlying function block instance. Two different `BoolForceable` instances at different memory addresses will NOT compare as equal, even if they hold references to the same hardware input. The duplicate check is object identity, not value equality.
