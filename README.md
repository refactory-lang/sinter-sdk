<p align="center">
  <a href="https://github.com/refactory-lang"><img src="https://raw.githubusercontent.com/refactory-lang/.github/main/assets/refactory-logo.svg" alt="Refactory" width="300"></a>
</p>

# sinter-sdk

Step SDKs for the Sinter compiled workflow automation platform. Operators write workflow steps in constrained Python or TypeScript, compiled to native Rust via the Refactory pipeline.

**Added in Refactory Supplement v0.3.**

## Dual-Language Support

| SDK | Language | Use Case |
|-----|----------|----------|
| `sinter-sdk-py` | Python | Step authoring for Python-native teams |
| `sinter-sdk-ts` | TypeScript | Step authoring for TS-native teams; n8n migration |

Both SDKs expose identical step protocols and types. Components written in either language compile to the same Rust runtime.

## Step Protocol

```python
# Python
from sinter_sdk import StepContext, StepResult
from dataclasses import dataclass
from returns.result import Result, Success

@dataclass(frozen=True)
class TransformStep:
    def execute(self, ctx: StepContext) -> Result[StepResult, str]:
        data = ctx.input_data()
        transformed = process(data)
        return Success(StepResult(transformed))
```

```typescript
// TypeScript
import { StepContext, StepResult } from '@sinter/sdk';
import { ok, Result } from 'neverthrow';

export function execute(ctx: StepContext): Result<StepResult, string> {
    const data = ctx.inputData();
    const transformed = process(data);
    return ok(new StepResult(transformed));
}
```

## License

Apache-2.0
