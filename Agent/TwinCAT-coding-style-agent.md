
========================
ARCHITECTURE MODEL
========================

EXEC_LAYER:
  FB_*:
    - cyclic execution only (task-driven)
    - owns ALL runtime control flow
    - ONLY place where logic loops exist
    - may instantiate and orchestrate DOMAIN objects
    - may contain system-level logic only

DOMAIN_LAYER:
  Class (no prefix):
    - passive objects only
    - NO cyclic execution
    - NO lifecycle ownership
    - NO autonomous behavior
    - reacts only when called by FB_*
    - must be fully stateless OR externally state-driven
    - must NOT contain scheduling logic

WIRING_LAYER:
  VAR / VAR_GLOBAL / VAR_INST:
    - compile-time dependency injection only
    - static composition only
    - no runtime instantiation

GLOBAL_RULE:
  - execution is exclusively owned by FB_*
  - domain objects are passive data/behavior containers
  - no hidden behavior allowed
  - all behavior must be explicitly triggered by FB_*

========================
TYPE SYSTEM
========================

I_      = Interface (full public contract boundary only)
ST_     = Struct
E_      = Enum
F_      = Function
FB_     = Function Block (execution unit only)
Class   = Domain object (no prefix, passive only)

RULE:
  - interfaces define full public contract
  - no partial/optional interface compliance

========================
CLASS RULES (DOMAIN OBJECTS)
========================

Class:
  passive = true
  cyclic_execution = false
  lifecycle_methods = forbidden
    (init/start/update/reset/execute/dispose)
  io_vars = forbidden
    (VAR_INPUT / VAR_OUTPUT / VAR_IN_OUT)
  fb_body_logic = forbidden (must remain empty)
  interface_required = true
  instantiation = compile-time only

METHOD_CONTRACT:
  - all behavior must be exposed via methods/properties
  - no hidden internal execution loops

========================
FB RULES (EXECUTION BLOCKS)
========================

FB_*:
  - only cyclic execution context allowed
  - owns control flow and timing
  - may contain:
      - domain object instances
      - orchestration logic
      - hardware interaction
  - must NOT behave as passive data container

========================
NAMING RULES
========================

METHODS / PROPERTIES:
  camelCase

PRIVATE / INTERNAL:
  _camelCase

BOOLEAN STYLE:
  isX / hasX / canX

CONSTANTS:
  UPPER_CASE_WITH_UNDERSCORES

ABBREVIATIONS:
  forbidden unless universally standard

ONE_CONCEPT_RULE:
  use one verb per concept consistently (get/fetch/retrieve etc)

========================
TYPE NAMING
========================

Interface: I_<Name>
Struct:     ST_<Name>
Enum:       E_<Name>
Function:   F_<Name>
FB:         FB_<Name>
Class:      <Name> (no prefix)

========================
HARDWARE SIGNALS
========================

PREFIXES:
  DI_ = digital input
  DO_ = digital output
  AI_ = analog input
  AO_ = analog output

RULES:
  - format: PREFIX_camelCase
  - no leading underscore allowed
  - must represent physical signal meaning (not logical abstraction)

========================
CONSTANTS + GENERIC VAR
========================

CONSTANTS:
  - UPPER_CASE_WITH_UNDERSCORES

VAR_GENERIC CONSTANT:
  allowed form:
    VAR_GENERIC CONSTANT
      <Name> : <Type> := <Value>
    END_VAR

RULE:
  - constants must not be computed at runtime

========================
INSTANCING RULES
========================

ALLOWED:
  VAR
  VAR_GLOBAL
  VAR_INST

FORBIDDEN:
  runtime allocation
  dynamic instantiation
  factory-based object creation

========================
PRAGMAS (TWINCAT-SPECIFIC)
========================

CLASS HEADER (MANDATORY WHEN CLASS USED):

  {attribute 'no_explicit_call' := '<Class> is a class, do not call directly, use methods'}
  {attribute 'hide_all_locals'}

RULE:
  - applies only to Class FUNCTION_BLOCK declarations
  - never used in FB_* execution blocks

HMI:

  VARIABLE:
    hmi : ST_<Name>_HMI

  PRAGMA:
    {attribute 'TcHmiSymbol.AddSymbol'}

HIDE INTERNAL OBJECTS:

  {attribute 'hide'}

RULE:
  - pragmas are metadata only
  - must never affect execution logic
  - must not be used for behavioral control

========================
TESTING CONVENTION
========================

CLASS TEST:
  C##_Name_TEST

TEST METHOD:
  T##_Description_ExpectedResult

STRUCTURE:
  VAR_INST:
    - expected / actual
  VAR:
    - SUT (system under test)

========================
FOLDER STRUCTURE (TwinCAT XAE)
========================

FunctionBlock/
  Public/
  Protected/
  Private/
    _stateMachine
    _logic
  I_Interface/
    Commands/
    Status/

RULE:
  - reflects visibility and separation of concerns
  - not optional for large modules

========================
DESIGN PRINCIPLES (ENFORCED)
========================

- execution is always explicit and FB-owned
- domain objects are passive and non-executing
- no hidden lifecycle or scheduling behavior
- no runtime instantiation
- no implicit side effects
- all interactions are statically wired or FB-triggered