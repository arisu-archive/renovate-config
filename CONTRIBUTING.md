# Contributing

This repo is org-wide critical infrastructure. Changes here propagate to every consumer repo on the next Renovate run.

## Before you open a PR

1. Read the relevant sub-preset and `README.md`. Understand which consumer repos a change touches.
2. Keep edits scoped to one sub-preset where possible. Cross-preset changes need an explicit reason in the PR description.
3. Run the validator locally:

   ```bash
   npx --yes --package renovate -- renovate-config-validator --strict
   ```

4. Validate every JSON file parses:

   ```bash
   find . -name '*.json' -not -path './node_modules/*' -exec python3 -m json.tool {} \;
   ```

## Adding a new sub-preset

1. Create `<name>.json` at the repo root with `$schema`, `description`, and the rules.
2. Add `"local>arisu-archive/renovate-config:<name>"` to the `extends` array in `default.json`.
3. Document it in the `README.md` Layout table.
4. Verify the validator passes.

## Naming a preset file

Use a full, descriptive lowercase noun for what the preset governs (`security.json`, `golang.json`). No abbreviations.

## Modifying an existing rule

- State the consumer-visible behavior change in the PR body (more PRs? fewer? slower automerge? new label?).
- If the change tightens policy (slower automerge, stricter pinning), it is safe to merge.
- If the change loosens policy (faster automerge, broader package matching, shorter soak), it requires a second platform reviewer.

## Local override etiquette for consumers

Encourage consumer repos to extend rather than copy-paste:

```json
{
  "extends": ["local>arisu-archive/renovate-config"],
  "packageRules": [{ "...repo-specific override here..." }]
}
```

Copy-paste forks of this config defeat the point of central management.

## Releases

This repo does not version preset releases. `extends: local>arisu-archive/renovate-config` always resolves to the default branch (`main`). Merge to `main` is the release. Branch protection on `main` is required.
