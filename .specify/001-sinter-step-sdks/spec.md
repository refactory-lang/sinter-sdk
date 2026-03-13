# Feature Specification: Sinter Step SDKs

**Feature Branch**: `001-sinter-step-sdks`
**Created**: 2026-03-13
**Status**: Draft

## Overview

The Sinter Step SDKs provide dual Python and TypeScript packages that define the `StepContext` and `StepResult` protocols for authoring workflow steps. Both language SDKs expose identical APIs so that step authors can work in their preferred language while producing components that translate cleanly to Rust for production execution.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Developer writes a workflow step in Python (Priority: P1)

As a workflow author, I want to write a Sinter step in Python that receives a `StepContext` and returns a `StepResult` so that I can process workflow data using Python libraries and have the step integrate into the Sinter execution engine.

**Why this priority**: Python step authoring is one of the two primary use cases and must work end-to-end.

**Independent Test**: Write a minimal step function that accepts `StepContext`, reads input data and config, and returns a `StepResult`. Run mypy and confirm zero type errors.

**Acceptance Scenarios**:

```
Scenario 1: Access input data from StepContext
  Given a StepContext with input_data containing {"orders": [{"id": 1, "total": 99.50}]}
  When the step function calls context.input_data["orders"]
  Then it receives the list of order dictionaries

Scenario 2: Access configuration from StepContext
  Given a StepContext with config containing {"batch_size": 100}
  When the step function calls context.config["batch_size"]
  Then it receives the integer 100

Scenario 3: Access secrets from StepContext
  Given a StepContext with secrets containing {"API_KEY": "sk-abc123"}
  When the step function calls context.secrets["API_KEY"]
  Then it receives the string "sk-abc123"

Scenario 4: Return success StepResult
  Given a step function that completes processing
  When it returns StepResult(output={"processed": 42}, status="success", error=None)
  Then the Sinter engine receives a successful result with the output data

Scenario 5: Return error StepResult
  Given a step function that encounters an error
  When it returns StepResult(output=None, status="error", error="Connection timeout after 30s")
  Then the Sinter engine receives a failed result with the error message
```

---

### User Story 2 - Developer writes a workflow step in TypeScript (Priority: P1)

As a workflow author, I want to write a Sinter step in TypeScript that receives a `StepContext` and returns a `StepResult` using the same API shape as the Python SDK so that my team can choose either language without learning a different interface.

**Why this priority**: TypeScript parity is essential; without it, TypeScript authors face a different mental model.

**Independent Test**: Write a minimal step function in TypeScript, run `tsc --strict`, and confirm zero type errors.

**Acceptance Scenarios**:

```
Scenario 1: TypeScript StepContext has identical shape to Python
  Given the TypeScript StepContext interface
  When I compare it to the Python StepContext protocol
  Then every property name and type is semantically equivalent

Scenario 2: TypeScript step compiles
  Given a step function typed as (context: StepContext) => StepResult
  When I run tsc --strict
  Then compilation succeeds with zero errors

Scenario 3: TypeScript step accesses input data
  Given a StepContext with inputData containing {"users": [{"name": "Alice"}]}
  When the step function accesses context.inputData.users
  Then it receives the array of user objects
```

---

### User Story 3 - Steps translate to Rust (Priority: P2)

As a platform engineer, I want steps authored in Python or TypeScript to translate to Rust via the refactory transpilation pipelines so that production execution avoids interpreter overhead.

**Why this priority**: Translation is downstream of authoring; authoring must be solid first.

**Independent Test**: Pass a sample Python step and a sample TypeScript step through the respective transpilation pipelines and verify the output compiles with `cargo check`.

**Acceptance Scenarios**:

```
Scenario 1: Python step translates
  Given a Python step using StepContext and StepResult
  When processed by the python-to-rust pipeline
  Then the output Rust file compiles and contains a function with matching signature

Scenario 2: TypeScript step translates
  Given a TypeScript step using StepContext and StepResult
  When processed by the typescript-to-rust pipeline
  Then the output Rust file compiles and contains a function with matching signature
```

---

### Edge Cases

- **Missing config key**: Accessing a non-existent key in `context.config` must raise `KeyError` (Python) / return `undefined` (TypeScript) rather than returning a silent null.
- **Empty input data**: `context.input_data` / `context.inputData` set to `{}` must not cause the step to crash; steps must handle empty input gracefully.
- **Large output payloads**: A `StepResult` with output exceeding 10 MB must be handled — the SDK should document maximum payload sizes or provide streaming alternatives.
- **Secret redaction**: When a `StepResult` error message accidentally contains a secret value, the SDK should not actively redact it (that is the engine's responsibility) but must never log secrets itself.
- **Naming convention bridge**: Python uses `snake_case` (`input_data`) while TypeScript uses `camelCase` (`inputData`). Translation pipelines must map between these correctly; the SDK must document the canonical field names for each language.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Python SDK MUST define a `StepContext` protocol with `input_data: dict[str, Any]`, `config: dict[str, Any]`, and `secrets: dict[str, str]` properties.
- **FR-002**: Python SDK MUST define a `StepResult` dataclass/protocol with `output: dict[str, Any] | None`, `status: Literal["success", "error"]`, and `error: str | None` fields.
- **FR-003**: TypeScript SDK MUST define a `StepContext` interface with `inputData: Record<string, unknown>`, `config: Record<string, unknown>`, and `secrets: Record<string, string>` properties.
- **FR-004**: TypeScript SDK MUST define a `StepResult` interface with `output: Record<string, unknown> | null`, `status: "success" | "error"`, and `error: string | null` fields.
- **FR-005**: The Python and TypeScript interfaces MUST be semantically identical — same property count, same property meanings, same type semantics after accounting for language-idiomatic naming.
- **FR-006**: Both SDKs MUST be independently installable (`pip install sinter-sdk` and `npm install @sinter/sdk`).
- **FR-007**: Python SDK MUST include `.pyi` stub files; TypeScript SDK MUST include `.d.ts` declaration files.
- **FR-008**: Both SDKs MUST export a `StepFunction` type alias representing the callable signature `(StepContext) -> StepResult`.
- **FR-009**: Both SDKs MUST validate that `status` is one of the allowed values at construction time.

### Key Entities

| Entity | Description |
|---|---|
| `StepContext` | Read-only container providing input data, configuration, and secrets to a step |
| `StepResult` | Return container holding step output, execution status, and optional error |
| `StepFunction` | Type alias for the step callable: receives `StepContext`, returns `StepResult` |

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A Python step using `StepContext` and `StepResult` passes `mypy --strict` with zero errors.
- **SC-002**: A TypeScript step using `StepContext` and `StepResult` passes `tsc --strict` with zero errors.
- **SC-003**: Property-by-property comparison of both SDKs shows 1:1 semantic equivalence (documented in a cross-reference table in the repo).
- **SC-004**: Sample steps in both languages translate to Rust via their respective pipelines and the output compiles with `cargo check`.
- **SC-005**: Both packages publish successfully to their respective registries (PyPI and npm) in CI.
- **SC-006**: SDK version numbers are kept in sync across Python and TypeScript releases.
