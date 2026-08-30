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

The workflow uses `continue-on-error: true` for the Action step because `FAIL` and `INCONCLUSIVE` are intentional test outcomes. A following assertion checks both the emitted `decision` and the GitHub Actions step `outcome`. `PASS` must produce a successful step; `FAIL` and `INCONCLUSIVE` must produce a failed Action step. The workflow is green only when both values match the matrix.

## Workflow

The workflow runs on pushes and manual dispatch:

- `.github/workflows/mcp-evidence-gate.yml` calls `yandexuanxuan/mcp-evidence-gate@b8cacb5eadca53c8b9a1e8d5c8ac956fd579238d` (the `v0.1.0-alpha.2` release commit).
- `dist/example-artifact.bin` is marked as binary in `.gitattributes` so Windows line-ending conversion cannot change its digest.
- Receipts live under `evidence/` and are intentionally small, deterministic fixtures.

This repository does not run a scanner and does not claim that the example server is safe. It verifies the downstream release-admission contract: scanner verdict, artifact binding, freshness, and policy decision remain distinct.
