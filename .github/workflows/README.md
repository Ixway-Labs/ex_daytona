# GitHub Actions Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| `test.yml` | push / PR | `mix check`-equivalent gate: tests + coverage floor (see `coveralls.json`), credo, dialyzer, minimum-toolchain compatibility, SBOM drift check |
| `conventional-commits.yml` | PR | Validates PR titles (Conventional Commits) — feeds release automation; use squash merges |
| `release.yml` | manual dispatch | git_ops release: version bump from commit history + changelog + tag, then triggers publish. First run tags the current version |
| `publish.yml` | `v*.*.*` tags | Publishes to Hex.pm and creates a GitHub release with the SBOM attached |
| `regenerate-sdk.yml` | spec changes on main | Opens a PR with the regenerated SDK |
| `breaking-changes.yml` | PRs touching spec/lib | oasdiff breaking-change gate + PR comment |

## Required secrets

- `HEX_API_KEY` — for publishing. Hex ≥ 2.5 removed
  `mix hex.user key generate`; create the key in the browser at
  hex.pm → Dashboard → Keys (permission `api:write`), then
  `gh secret set HEX_API_KEY`.

## Notes

- Spec refresh is manual: run `./scripts/fetch-spec.sh` and push the result
  (the weekly spec-sync workflow was removed; it had been failing every run).
- PRs created by workflows with the default `GITHUB_TOKEN`
  (regenerate-sdk) do **not** trigger other workflows. For CI on those PRs,
  store a fine-grained PAT as a secret and use it as the `token` input of
  the create-pull-request steps.
- release.yml never uses `git_ops.release --initial` (it conflicts with
  the existing CHANGELOG.md); the first release tags the current
  `@version` directly. Do the same when releasing locally.
