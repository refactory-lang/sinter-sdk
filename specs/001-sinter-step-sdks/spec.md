# Feature Specification: Sinter Step SDKs (Python & TypeScript)

**Feature Branch**: `001-sinter-step-sdks`
**Created**: 2026-03-13
**Status**: Draft
**Input**: User description: "Implement Sinter Step SDKs for Python and TypeScript - StepContext and StepResult protocols, dual-language parity, type definitions for workflow step authoring"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Write a Workflow Step in Python (Priority: P1)

A Python developer authors a Sinter workflow step by importing `StepContext` and `StepResult` from the `sinter_sdk` package. They define a step class with an `execute` method that receives a `StepContext`, reads input data and configuration from it, performs a transformation, and returns a `StepResult` wrapped in a `Result` type from the `returns` library. The step passes `mypy --strict` type checking without errors.

**Why this priority**: Python step authoring is the core use case. Without a working Python SDK, no steps can be written or compiled through the Python-to-Rust pipeline. This is the minimum viable product.

**Independent Test**: Can be fully tested by creating a sample step file that imports `StepContext` and `StepResult`, implements `execute`, and running `mypy --strict` against it. A passing type check and a successful unit test of the execute method confirms the SDK works.

**Acceptance Scenarios**:

1. **Given** a Python file that imports `StepContext` and `StepResult` from `sinter_sdk`, **When** `mypy --strict` is run against the file, **Then** no type errors are reported.
2. **Given** a step class with an `execute(self, ctx: StepContext) -> Result[StepResult, str]` method, **When** `ctx.input_data()` is called, **Then** it returns the step's input payload as a typed dictionary.
3. **Given** a step that calls `ctx.config("timeout")`, **When** the config key exists, **Then** the value is returned as `str`; when missing, **Then** `None` is returned.
4. **Given** a step that calls `ctx.secret("api_key")`, **When** the secret is available, **Then** its value is returned; when unavailable, **Then** the result is `None` and no exception is raised.

---

### User Story 2 - Write a Workflow Step in TypeScript (Priority: P1)

A TypeScript developer authors a Sinter workflow step by importing `StepContext` and `StepResult` from `@sinter/sdk`. They export an `execute` function that receives a `StepContext`, accesses input data, config, and secrets, and returns a `Result<StepResult, string>` using the `neverthrow` library. The step passes `tsc --strict` without errors.

**Why this priority**: TypeScript parity is co-equal with Python. Teams migrating from n8n need TypeScript support from day one. Both SDKs must ship together to deliver on the dual-language promise.

**Independent Test**: Can be fully tested by writing a sample `.ts` step file, compiling with `tsc --strict`, and running a unit test that invokes `execute` with a mock `StepContext`.

**Acceptance Scenarios**:

1. **Given** a TypeScript file that imports `StepContext` and `StepResult` from `@sinter/sdk`, **When** `tsc --strict` is run, **Then** no type errors are reported.
2. **Given** an `execute` function with signature `(ctx: StepContext) => Result<StepResult, string>`, **When** `ctx.inputData()` is called, **Then** it returns the input payload as `Record<string, unknown>`.
3. **Given** a step that calls `ctx.config("timeout")`, **When** the key exists, **Then** the value is returned as `string`; when missing, **Then** `undefined` is returned.
4. **Given** a step that calls `ctx.secret("api_key")`, **When** the secret is available, **Then** its value is returned; when unavailable, **Then** `undefined` is returned and no exception is thrown.

---

### User Story 3 - Dual-Language API Parity (Priority: P2)

A team lead evaluates the Sinter SDKs and confirms that both SDKs expose the same capabilities with equivalent semantics. Every method on `StepContext` in Python has a corresponding method on `StepContext` in TypeScript (adjusted for language naming conventions: `snake_case` in Python, `camelCase` in TypeScript). The `StepResult` constructors accept the same logical fields. A step written in Python can be rewritten in TypeScript (or vice versa) with a mechanical translation and no loss of functionality.

**Why this priority**: Parity is essential for the Sinter compilation model where both languages target the same Rust runtime. Semantic divergence between SDKs would cause runtime inconsistencies.

**Independent Test**: Create a parity test matrix that lists every method on `StepContext` and `StepResult` in both languages and verifies 1:1 correspondence. Write the same step logic in both languages and assert identical outputs given the same inputs.

**Acceptance Scenarios**:

1. **Given** the Python SDK's `StepContext` with methods `input_data()`, `config(key)`, `secret(key)`, and `step_id()`, **When** the TypeScript SDK is inspected, **Then** it exposes `inputData()`, `config(key)`, `secret(key)`, and `stepId()` with equivalent signatures and return types.
2. **Given** the Python SDK's `StepResult` accepting `data`, `status`, and optional `error`, **When** the TypeScript SDK's `StepResult` is inspected, **Then** it accepts the same fields with equivalent types.
3. **Given** a step implemented in both languages with identical logic, **When** both are executed with the same input context, **Then** the resulting `StepResult` payloads are semantically identical.

---

### User Story 4 - Steps Translate Through Compilation Pipeline (Priority: P2)

A developer writes a step in Python (or TypeScript) and submits it to the Refactory compilation pipeline. The pipeline's type extractor reads the SDK type annotations, maps them to the Rust target types, and produces a compiled Rust step. The compiled step preserves the same `StepContext` access patterns and `StepResult` output shape.

**Why this priority**: The SDK exists to feed the compilation pipeline. If SDK types do not translate cleanly, the entire Sinter platform breaks. This validates end-to-end correctness.

**Independent Test**: Run the type extractor on a sample step file and verify the output Rust struct definitions match expected shapes. This can be tested with snapshot tests against the extractor output.

**Acceptance Scenarios**:

1. **Given** a Python step using `StepContext.input_data()` and returning `StepResult(data={"key": "value"})`, **When** the python-to-rust pipeline processes the file, **Then** the output Rust code contains a function that reads from a `StepContext` struct and returns a `StepResult` struct with matching field names.
2. **Given** a TypeScript step using `ctx.inputData()` and returning `ok(new StepResult({key: "value"}))`, **When** the ts-to-rust pipeline processes the file, **Then** the output Rust code is structurally equivalent to the Python pipeline output.

---

### User Story 5 - Error Handling in Steps (Priority: P3)

A developer writes a step that encounters an error during execution (e.g., invalid input data, missing required config). They return an error result using the language-appropriate `Result` type (`Failure` in Python via `returns`, `err` in TypeScript via `neverthrow`). The error message is preserved in the `StepResult` and propagated through the pipeline without loss.

**Why this priority**: Error handling is critical for production workflows but is not needed for initial SDK validation. The happy path must work first.

**Independent Test**: Write a step that returns an error result, invoke it in a unit test, and assert the error message is accessible and correctly typed.

**Acceptance Scenarios**:

1. **Given** a Python step that returns `Failure("input missing field 'email'")`, **When** the result is inspected, **Then** it is a `Failure` instance containing the error string.
2. **Given** a TypeScript step that returns `err("input missing field 'email'")`, **When** the result is inspected, **Then** it is an `Err` instance containing the error string.
3. **Given** an error result from either language, **When** processed by the pipeline, **Then** the error message appears in the compiled Rust step's error variant.

---

### Edge Cases

- What happens when `ctx.input_data()` is called but the step received no input? The SDK must return an empty dictionary/object, not raise an exception or return null.
- What happens when `ctx.config(key)` is called with a key that contains special characters (dots, slashes)? The SDK must treat keys as opaque strings and return `None`/`undefined` if not found.
- What happens when `ctx.secret(key)` is called in a test environment where no secret store is configured? The SDK must return `None`/`undefined` and not crash.
- What happens when `StepResult` is constructed with `data=None` (Python) or `data: undefined` (TypeScript)? The SDK must accept this and produce a result with empty/null data.
- What happens when a step returns neither `Success`/`ok` nor `Failure`/`err` (e.g., throws an unhandled exception)? The pipeline must treat this as an unrecoverable step failure and surface the exception message.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The Python SDK MUST export `StepContext` and `StepResult` from the `sinter_sdk` top-level package via `__init__.py`.
- **FR-002**: The TypeScript SDK MUST export `StepContext` and `StepResult` from the `@sinter/sdk` package entry point.
- **FR-003**: `StepContext` MUST provide a method to retrieve the step's input data as a dictionary (Python: `dict[str, Any]`) or record (TypeScript: `Record<string, unknown>`).
- **FR-004**: `StepContext` MUST provide a method to retrieve a configuration value by key, returning `None`/`undefined` when the key is absent.
- **FR-005**: `StepContext` MUST provide a method to retrieve a secret value by key, returning `None`/`undefined` when the key is absent, without raising exceptions or throwing errors.
- **FR-006**: `StepContext` MUST provide a method to retrieve the current step's unique identifier as a string.
- **FR-007**: `StepResult` MUST accept output data (dictionary/record), a status string, and an optional error string.
- **FR-008**: `StepResult.status` MUST be constrained to a known set of values: `"success"`, `"failure"`, `"skipped"`.
- **FR-009**: The Python SDK MUST use `returns.result.Result[StepResult, str]` as the return type for step execution.
- **FR-010**: The TypeScript SDK MUST use `neverthrow.Result<StepResult, string>` as the return type for step execution.
- **FR-011**: Both SDKs MUST pass strict type checking (`mypy --strict` for Python, `tsc --strict` for TypeScript) with zero errors.
- **FR-012**: Python naming conventions MUST use `snake_case` for all methods and fields. TypeScript naming conventions MUST use `camelCase`.
- **FR-013**: SDK types MUST be annotated with sufficient type information for the Refactory pipeline type extractor to map them to Rust equivalents without ambiguity.
- **FR-014**: `StepContext` MUST be immutable/read-only. Steps must not be able to mutate the context.
- **FR-015**: `StepResult` MUST be immutable after construction. In Python, this means using `@dataclass(frozen=True)` or equivalent. In TypeScript, this means using `readonly` fields.

### Key Entities

- **StepContext**: Represents the runtime environment provided to a step during execution. Provides read-only access to input data (the payload from the previous step or trigger), configuration values (key-value pairs set at workflow design time), secrets (sensitive values resolved at runtime from a secret store), and step metadata (step ID). Cannot be constructed by step authors; it is injected by the Sinter runtime.
- **StepResult**: Represents the output of a step execution. Contains the output data payload (passed to the next step in the workflow), a status indicator (`success`, `failure`, `skipped`), and an optional error message. Constructed by step authors and returned from the `execute` function/method.
- **Step**: A unit of work in a Sinter workflow. Defined by step authors as a class (Python) or function (TypeScript) that receives a `StepContext` and returns a `Result` wrapping a `StepResult`. Steps are the compilation unit for the Refactory pipeline.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A sample Python step importing `StepContext` and `StepResult` from `sinter_sdk` passes `mypy --strict` with zero errors.
- **SC-002**: A sample TypeScript step importing `StepContext` and `StepResult` from `@sinter/sdk` passes `tsc --strict` with zero errors.
- **SC-003**: 100% of `StepContext` methods in the Python SDK have a corresponding method in the TypeScript SDK with equivalent semantics (verified by a parity matrix).
- **SC-004**: 100% of `StepResult` fields in the Python SDK have a corresponding field in the TypeScript SDK with equivalent types.
- **SC-005**: A Python step and a TypeScript step implementing identical logic produce semantically identical `StepResult` outputs when given the same `StepContext` inputs.
- **SC-006**: The Refactory type extractor can process SDK type annotations from both languages and produce valid Rust struct definitions without manual intervention.
- **SC-007**: Unit tests for both SDKs achieve 100% line coverage of `StepContext` and `StepResult` implementations.
- **SC-008**: Both SDKs have zero runtime dependencies beyond `returns` (Python) and `neverthrow` (TypeScript) for Result types.

---

## v0.3 Addendum: Milestone 2 Track B — Workflow Runtime Context

*Added 2026-03-16 to align with master spec v0.3 §9.5*

### Workflow Schema

This SDK is part of **Milestone 2 Track B** (Sinter Core). The workflow runtime uses a YAML/JSON workflow schema that defines step composition, input/output connections, and trigger configuration. The SDK must define types that are compatible with this schema.

### Runtime Semantics

- The Sinter workflow runtime is built on **Rust with Tokio** for async execution
- Steps may be async; the runtime manages concurrency, timeouts, and cancellation
- `StepContext` instances are constructed by the runtime, not by step authors — the SDK defines the interface contract

### Additional Requirements

- **FR-019**: The SDK MUST define TypeScript type declarations for the workflow YAML/JSON schema (step definitions, input/output bindings, trigger configuration)
- **FR-020**: Steps authored with the SDK MUST be compilable through the Refactory transformation pipeline (python-to-rust or typescript-to-rust) to native Rust
- **FR-021**: The SDK MUST document forward/backward compatibility guarantees between SDK versions and Sinter runtime versions

### Additional Success Criteria

- **SC-009**: A 5-step workflow defined in YAML with 2+ connectors compiles to a native binary via the Refactory pipeline
- **SC-010**: Performance benchmarks show measurable improvement over equivalent n8n workflow execution
