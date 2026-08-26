# mcp-evidence-gate-dogfood

Cross-repository dogfood for the [MCP Evidence Gate](https://github.com/yandexuanxuan/mcp-evidence-gate) GitHub Action.

This repository is an executable specification for downstream consumers. It keeps a small artifact and receipt fixtures, invokes the Action by immutable commit SHA, and asserts the expected release decision for each case.

## Decision matrix

| Case | Expected decision | Meaning |
| --- | --- | --- |
| matching clean receipt | `PASS` | The receipt binds to the artifact and satisfies the permissive policy. |
| digest mismatch | `INCONCLUSIVE` | The evidence cannot support this artifact; this is not a server-safety claim. |
| stale receipt | `INCONCLUSIVE` | The evidence freshness window has expired. |
| findings receipt | `FAIL` | The scanner reported findings. |
| malformed receipt | `FAIL` | The receipt fails structural conformance. |

The workflow uses `continue-on-error: true` for the Action step because `FAIL` and `INCONCLUSIVE` are intentional test outcomes. A following assertion checks the emitted `decision`, so the workflow remains green only when fail-closed behavior is correct.

## Workflow

The workflow runs on pushes and manual dispatch:

- `.github/workflows/mcp-evidence-gate.yml` calls `yandexuanxuan/mcp-evidence-gate@85943698f82639ec40e19a57e2ee74a0958574b0`.
- `dist/example-artifact.bin` is marked as binary in `.gitattributes` so Windows line-ending conversion cannot change its digest.
- Receipts live under `evidence/` and are intentionally small, deterministic fixtures.

This repository does not run a scanner and does not claim that the example server is safe. It verifies the downstream release-admission contract: scanner verdict, artifact binding, freshness, and policy decision remain distinct.
