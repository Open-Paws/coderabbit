# AGENTS.md — Open-Paws/coderabbit

This repo holds the **org-wide default CodeRabbit configuration** for the Open Paws GitHub organization. Any Open Paws repository that does not define its own `.coderabbit.yaml` automatically inherits the config here. Changes to this repo affect every PR reviewed across the entire organization.

---

## Status

**Maintenance** — The config is stable. Changes here are high-impact: a typo or invalid key silently breaks CodeRabbit reviews org-wide. Treat every edit with the same care as a change to shared CI infrastructure.

---

## Key files

| File | Purpose |
|---|---|
| `.coderabbit.yaml` | Primary config — inherited by all repos without their own config |
| `semgrep-no-animal-violence.yaml` | Imports the full Open Paws speciesist-language rule set for Semgrep |
| `.pre-commit-config.yaml` | Pre-commit hook config for local development (NAV check) |
| `.github/workflows/auto-merge.yml` | Wave 0 auto-merge gate — deploy to target repos via scripts |
| `.github/workflows/no-animal-violence.yml` | NAV CI check — runs on every PR in this repo |

---

## Config structure summary

The `.coderabbit.yaml` is organized into five sections:

1. **`reviews`** — profile, auto-review triggers, path filters, path-specific instructions, pre-merge checks, finishing touches, and tool integrations
2. **`chat`** — auto-reply and UI settings
3. **`knowledge_base`** — controls what CodeRabbit reads and learns; scoped locally per repo; web search disabled
4. **`code_generation`** — unit test generation instructions
5. **Top-level keys** — `language` and `tone_instructions` (movement terminology required)

---

## How to test config changes

CodeRabbit validates `.coderabbit.yaml` against its JSON schema on load. To check locally before pushing:

```bash
# Install a YAML schema validator
pip install check-jsonschema

# Validate against the CodeRabbit v2 schema
check-jsonschema \
  --schemafile https://coderabbit.ai/integrations/schema.v2.json \
  .coderabbit.yaml
```

After merging to `main`, open a test PR in any repo that inherits this config and confirm CodeRabbit posts a review summary. Check that:

- The high-level summary appears
- Pre-merge checks run (visible in the review comment)
- Semgrep/TruffleHog tool results appear (if the PR touches relevant files)
- No schema validation errors appear in the CodeRabbit comment

There is no staging environment for CodeRabbit config — test in a low-risk repo (e.g. `coderabbit` itself or `desloppify`) before relying on the change org-wide.

---

## Integration points

This config is the baseline for **all Open Paws repositories**. It is actively used by:

- `platform` — Astro 5 + React 19 + Supabase frontend/backend
- `gary` — autonomous agent (overrides with `knowledge_base.opt_out: true`)
- `graze-cli` — agentic coding CLI
- `desloppify` — code quality scanner
- `project-compassionate-code` — org-scale PR submission tool
- All other repos in the Open-Paws org that lack their own `.coderabbit.yaml`

The `auto-merge.yml` workflow is designed to be deployed to target repos (not run from here). The NAV GitHub Action and pre-commit hook in this repo serve as the reference implementation for all other repos.

---

## Safe vs. risky changes

### Safe to change

- `tone_instructions` — cosmetic; affects review wording only
- `reviews.poem` / `chat.art` — UI preferences
- `reviews.path_filters` — adding new exclusion patterns
- `code_generation.unit_tests.path_instructions` — test generation guidance

### Requires care

- `reviews.profile` — changing from `assertive` to `chill` or `nitpick` alters review volume org-wide
- `reviews.path_instructions` — modifies what CodeRabbit checks on specific file types for every repo
- `reviews.pre_merge_checks` — adding/removing/changing error-mode checks directly gates merges org-wide
- `knowledge_base.opt_out` — setting to `true` here disables the KB for all Tier 1 repos (not usually desired; use per-repo override instead)
- `reviews.tools` — disabling Semgrep or TruffleHog removes security scanning across the org

### Never do

- Remove the `no speciesist idioms` pre-merge check — this is a core Open Paws standard
- Remove the `no hardcoded secrets` pre-merge check — this is a security baseline
- Set `reviews.auto_review.enabled: false` without an explicit decision to disable automated review org-wide
- Commit investigation names, witness identifiers, or activist identities to this repo — it is public

---

## Tier-based overrides for sensitive repos

Investigation data, witness information, and coalition operations repos (Tier 2/3) must override the knowledge base opt-out to prevent sensitive data from entering CodeRabbit's learning corpus:

```yaml
# Add to the repo-level .coderabbit.yaml
inheritance: true

knowledge_base:
  opt_out: true
```

Tier classifications are maintained in `ecosystem/repos.md` in the [Open-Paws/context](https://github.com/Open-Paws/context) strategy repo.

---

## TODOs

- [ ] Re-enable `knowledge_base.mcp.usage` once the Gary MCP hub is active (currently `disabled`)
- [ ] Add `betterleaks` tool once it appears in `schema.v2.json` (was removed in a prior fix commit — see `d8fd4a9`)
- [ ] Create `scripts/deploy-auto-merge-workflow.sh` referenced in `auto-merge.yml` comments
- [ ] Add schema validation to the CI pipeline for this repo itself (currently no workflow validates `.coderabbit.yaml` on PRs)
- [ ] Document the full list of repos that have opted out of inheritance with their own `.coderabbit.yaml`
