# renovate-config

Central [Renovate](https://docs.renovatebot.com) configuration presets for `arisu-archive`. One source of truth — every repo extends the same default preset, so dependency hygiene, automerge policy, security handling, and grouping stay consistent across the org.

## Usage

In any repo, drop a `renovate.json` at the repo root:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>arisu-archive/renovate-config"]
}
```

That single line pulls in `default.json` from this repo and all composed sub-presets.

## Override per repo

Local overrides go after the extends. Local rules win.

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>arisu-archive/renovate-config"],
  "timezone": "UTC",
  "packageRules": [
    {
      "matchPackageNames": ["some-fragile-package"],
      "enabled": false
    }
  ]
}
```

## Cherry-pick a single sub-preset

```json
{
  "extends": [
    "local>arisu-archive/renovate-config:base",
    "local>arisu-archive/renovate-config:security"
  ]
}
```

## Layout

| File | Purpose |
|---|---|
| `default.json` | Entrypoint. Composes every sub-preset. Extend this from consumer repos. |
| `base.json` | Timezone, concurrency limits, dependency dashboard, labels, PR body, rebase policy. |
| `schedule.json` | Weekly Monday early-morning schedule (Asia/Hong_Kong); lockfile maintenance. |
| `grouping.json` | Group rules. Bundles non-major updates; isolates linters, test frameworks, type defs. |
| `automerge.json` | Patch + minor automerge after CI; majors require manual review; high-risk packages excluded. |
| `security.json` | OSV-backed vulnerability alerts. Bypasses schedule and soak window. High priority. |
| `golang.json` | `gomodTidy`, indirect-dep grouping, strict toolchain constraint filtering. |
| `docker.json` | Pinned digests; groups image updates; trusts distroless/chainguard with shorter soak. |
| `github-actions.json` | SHA-pinned actions (supply-chain hardening), grouped non-major updates. |

## Policy summary

| Policy | Value | Where |
|---|---|---|
| Timezone | `Asia/Hong_Kong` | `base.json` |
| Schedule | `before 6am on monday` | `schedule.json` |
| Soak window (default) | 7 days | `automerge.json` |
| Soak window (third-party direct) | 14 days | `automerge.json` |
| Soak window (security) | 0 days, immediate | `security.json` |
| Major updates | Manual review only | `automerge.json` |
| Concurrent PRs | 5 | `base.json` |
| Concurrent branches | 10 | `base.json` |
| Hourly PR creation cap | 4 | `base.json` |
| Pinned digests | GitHub Actions, Docker | `github-actions.json`, `docker.json` |
| Vulnerability source | OSV + GitHub advisories | `security.json` |
| Config migration PRs | Enabled | `default.json`, `base.json` |
| Commit style | Conventional / `:semanticCommits` | `default.json` |

## Onboarding a new repo

1. Install the Renovate GitHub App (or self-hosted runner) on the repo.
2. Add the `renovate.json` shown in [Usage](#usage).
3. Merge the onboarding PR Renovate opens.
4. Approve the first Dependency Dashboard issue Renovate creates.

## CI

`.github/workflows/validate.yml` runs on every push and PR:

- `renovate-config-validator --strict` — fails on unknown keys, deprecated options, schema drift.
- JSON syntax lint across every `*.json` in the repo.

## Modifying a preset

1. Edit the relevant sub-preset file. Keep changes scoped to the preset's stated purpose.
2. Verify locally:
   ```bash
   npx --yes --package renovate -- renovate-config-validator --strict
   ```
3. Open a PR. CI must pass.
4. After merge, every consumer repo picks the change up on their next Renovate run.

## References

- [Renovate docs](https://docs.renovatebot.com)
- [Configuration options](https://docs.renovatebot.com/configuration-options/)
- [Presets](https://docs.renovatebot.com/config-presets/)
- [Best practices](https://docs.renovatebot.com/upgrade-best-practices/)
- [OSV vulnerability alerts](https://docs.renovatebot.com/configuration-options/#osvvulnerabilityalerts)

## License

[MIT](./LICENSE)
