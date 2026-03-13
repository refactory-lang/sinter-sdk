# Requirements Checklist: Sinter Step SDKs

**Feature Branch**: `001-sinter-step-sdks`
**Last Updated**: 2026-03-13

## Functional Requirements

### Python SDK - StepContext

- [ ] **FR-001**: `StepContext` and `StepResult` exported from `sinter_sdk` top-level `__init__.py`
- [ ] **FR-003**: `StepContext.input_data()` returns `dict[str, Any]`; returns empty dict when no input provided
- [ ] **FR-004**: `StepContext.config(key: str)` returns `Optional[str]`; returns `None` for missing keys
- [ ] **FR-005**: `StepContext.secret(key: str)` returns `Optional[str]`; returns `None` for missing keys without raising
- [ ] **FR-006**: `StepContext.step_id()` returns `str` with the current step's unique identifier
- [ ] **FR-014**: `StepContext` is immutable (no public setters, no mutable state)

### Python SDK - StepResult

- [ ] **FR-007**: `StepResult` accepts `data: dict[str, Any]`, `status: StepStatus`, and `error: Optional[str]`
- [ ] **FR-008**: `StepResult.status` constrained to `"success"`, `"failure"`, `"skipped"` via enum or literal type
- [ ] **FR-015**: `StepResult` is immutable after construction (`@dataclass(frozen=True)` or equivalent)

### Python SDK - Type Safety

- [ ] **FR-009**: Step execute return type is `Result[StepResult, str]` from the `returns` library
- [ ] **FR-011**: All SDK source files pass `mypy --strict` with zero errors
- [ ] **FR-012**: All Python methods and fields use `snake_case` naming

### TypeScript SDK - StepContext

- [ ] **FR-002**: `StepContext` and `StepResult` exported from `@sinter/sdk` package entry point
- [ ] **FR-003**: `StepContext.inputData()` returns `Record<string, unknown>`; returns empty object when no input provided
- [ ] **FR-004**: `StepContext.config(key: string)` returns `string | undefined`; returns `undefined` for missing keys
- [ ] **FR-005**: `StepContext.secret(key: string)` returns `string | undefined`; returns `undefined` for missing keys without throwing
- [ ] **FR-006**: `StepContext.stepId()` returns `string` with the current step's unique identifier
- [ ] **FR-014**: `StepContext` is immutable (`readonly` properties, no mutation methods)

### TypeScript SDK - StepResult

- [ ] **FR-007**: `StepResult` accepts `data: Record<string, unknown>`, `status: StepStatus`, and optional `error: string`
- [ ] **FR-008**: `StepResult.status` constrained to `"success" | "failure" | "skipped"` via union type
- [ ] **FR-015**: `StepResult` is immutable after construction (`readonly` fields)

### TypeScript SDK - Type Safety

- [ ] **FR-010**: Step execute return type is `Result<StepResult, string>` from the `neverthrow` library
- [ ] **FR-011**: All SDK source files pass `tsc --strict` with zero errors
- [ ] **FR-012**: All TypeScript methods and fields use `camelCase` naming

### Cross-Language Parity

- [ ] **FR-013**: SDK types carry sufficient annotations for Refactory pipeline type extraction
- [ ] Every `StepContext` method in Python has a corresponding method in TypeScript
- [ ] Every `StepResult` field in Python has a corresponding field in TypeScript
- [ ] Naming conventions follow language idioms (`snake_case` vs `camelCase`) while preserving semantic equivalence

## Edge Cases

- [ ] `ctx.input_data()` / `ctx.inputData()` returns empty dict/object when step received no input
- [ ] `ctx.config(key)` handles keys with special characters (dots, slashes) as opaque strings
- [ ] `ctx.secret(key)` returns `None`/`undefined` gracefully when no secret store is configured
- [ ] `StepResult` accepts `None`/`undefined` data without error
- [ ] Unhandled exceptions in step execution are surfaced as pipeline failures with the exception message

## Success Criteria

- [ ] **SC-001**: Sample Python step passes `mypy --strict`
- [ ] **SC-002**: Sample TypeScript step passes `tsc --strict`
- [ ] **SC-003**: 100% StepContext method parity between languages (verified by parity matrix)
- [ ] **SC-004**: 100% StepResult field parity between languages
- [ ] **SC-005**: Identical logic in both languages produces identical outputs
- [ ] **SC-006**: Refactory type extractor processes SDK annotations from both languages
- [ ] **SC-007**: Unit tests achieve 100% line coverage of StepContext and StepResult
- [ ] **SC-008**: Zero runtime dependencies beyond `returns` (Python) and `neverthrow` (TypeScript)
