<!-- codemod-skill-discovery:begin -->
## Codemod Skill Discovery
This section is managed by `codemod` CLI.

- Core skill: `.agents/skills/codemod/SKILL.md`
- Package skills: `.agents/skills/<package-skill>/SKILL.md`
- List installed Codemod skills: `npx codemod agent list --harness antigravity --format json`

<!-- codemod-skill-discovery:end -->

## Project: sinter-sdk

Step SDKs for the Sinter compiled workflow automation platform. Part of the [refactory-lang](https://github.com/refactory-lang) organization. Operators write workflow steps in constrained Python or TypeScript; both compile to the same native Rust runtime via the Refactory pipeline. Added in Refactory Supplement v0.3.

### Architecture

- **Python SDK** (`python/sinter_sdk/`): Python step authoring types and protocols (`StepContext`, `StepResult`)
- **TypeScript SDK** (`typescript/src/`): TypeScript step authoring types (`@sinter/sdk`)
- **Tests** (`tests/`): Shared test suite
- **Specs** (`specs/`): Implementation specifications

Both SDKs expose identical step protocols. The step protocol is: implement an `execute` method that takes a `StepContext` and returns a `Result[StepResult, str]`.

### Running

```bash
# Python SDK
cd python && pip install -e . && pytest

# TypeScript SDK
cd typescript && npm install && npm test
```

### Key Files

| File | Purpose |
|------|---------|
| `python/sinter_sdk/` | Python SDK package (`StepContext`, `StepResult`, step protocols) |
| `typescript/src/` | TypeScript SDK source (`@sinter/sdk`) |
| `tests/` | Shared test suite |
| `specs/` | Implementation specifications |

### Conventions

- Both SDKs must maintain API parity -- identical step protocols and types
- Python uses `returns.result` for Result types; TypeScript uses `neverthrow`
- Components compile to the same Rust runtime regardless of source language
