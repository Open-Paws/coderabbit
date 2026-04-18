# Open Paws — CodeRabbit Central Config

![Maintenance](https://img.shields.io/badge/status-maintenance-blue)
[![Open Paws Ecosystem](https://img.shields.io/badge/ecosystem-Open%20Paws-green)](https://github.com/Open-Paws)

Org-wide default CodeRabbit configuration for every repository in the Open Paws GitHub organization.

---

## What is CodeRabbit?

[CodeRabbit](https://coderabbit.ai) is an AI-powered code review tool that integrates with GitHub pull requests. It reads a `.coderabbit.yaml` configuration file and applies automated review logic — flagging issues, suggesting fixes, scanning for security problems, and enforcing team standards — on every PR.

Open Paws runs CodeRabbit at the **org level**: a single configuration in this repo applies automatically to every Open Paws repository that does not define its own `.coderabbit.yaml`. This keeps the review baseline consistent across all campaigns, investigation tooling, platform code, and agent infrastructure.

---

## How inheritance works

| Repo state | Effective config |
|---|---|
| No `.coderabbit.yaml` in the repo | This central config applies automatically |
| Has `.coderabbit.yaml` **without** `inheritance: true` | Repo config fully replaces this config |
| Has `.coderabbit.yaml` **with** `inheritance: true` at the top | Repo config merges on top of this config |

To extend rather than replace, add `inheritance: true` as the first line of the repo-level `.coderabbit.yaml`.

---

## What this config sets

### Review profile

- **Assertive** profile — flags issues without being overly verbose
- `request_changes_workflow: true` — CodeRabbit can block merges when it finds errors; required for the Wave 0 auto-merge gate
- High-level PR summary enabled; poem disabled; effort estimation enabled
- Auto-incremental reviews on each push; no review on draft PRs
- Ignores `dependabot[bot]` and `renovate[bot]` commits
- Skips reviews for PRs with `WIP` or `DO NOT MERGE` in the title

### Automatic labeling

CodeRabbit suggests (but does not auto-apply) two labels:

- `security` — changes touching authentication, secrets, or coalition/activist data
- `breaking-change` — removals or alterations of public API contracts, endpoints, or schemas

### Path filters

The following are excluded from review:

- `dist/**`, `node_modules/**`
- `**/*.lock`, `**/*.generated.*`
- `.claude/worktrees/**`

### Path-specific review instructions

| Path pattern | What CodeRabbit checks |
|---|---|
| `**/*.test.{ts,js,py}` | Three-question test quality check: would it fail on breakage? does it encode a domain rule? would mutation testing kill it? Rejects snapshot traps, coverage theater, and happy-path-only tests. |
| `.claude/**` | Scans for hidden Unicode (Rules File Backdoor attack). Flags hooks that expand scope. Confirms no activist identities or investigation names are embedded in instruction files. |

### Pre-merge checks (hard errors)

Two checks run as **errors** — they block merge:

1. **No hardcoded secrets** — fails if any non-test file contains API keys, tokens, passwords, database URLs, or service account credentials
2. **No speciesist idioms** — fails if code, comments, variable names, or docs contain terms from the no-animal-violence pattern list (e.g. `livestock` → `farmed animals`, `master/slave` → `primary/replica`, `whitelist/blacklist` → `allowlist/denylist`)

PR title format is checked as a **warning** (imperative verb, under 70 characters).

### Finishing touches

After each review CodeRabbit offers:

- Unit test suggestions for uncovered code
- Inline fixes for any speciesist idiom violations found in changed files

### Static analysis tools

| Tool | Purpose |
|---|---|
| Semgrep | Runs `semgrep-no-animal-violence.yaml` — imports the full Open Paws [no-animal-violence rule set](https://github.com/Open-Paws/semgrep-rules-no-animal-violence) |
| TruffleHog | Secret scanning on every PR |
| GitHub Checks | CI status integration (90 s timeout) |
| LanguageTool | Default-level prose linting |

### Knowledge base

- Reads `CLAUDE.md` and `AGENTS.md` files in each repo as code guidelines
- Learnings, issues, and PRs are scoped **locally** per repo — no cross-repo leakage
- Web search disabled
- Jira, Linear, and MCP integrations disabled (MCP re-enable pending Gary MCP hub activation)

### Chat

- Auto-reply to `@coderabbitai` comments enabled
- Decorative ASCII art disabled

---

## Open Paws-specific standards enforced

### Compassionate language (no-animal-violence)

CodeRabbit hard-fails any PR that introduces speciesist idioms into code, comments, or documentation. Key mappings enforced:

| Avoid | Use instead |
|---|---|
| `livestock` | `farmed animals` |
| `master` / `slave` | `primary` / `replica` |
| `whitelist` / `blacklist` | `allowlist` / `denylist` |
| `cattle vs. pets` | `ephemeral vs. persistent` |
| `kill two birds with one stone` | `accomplish two things at once` |
| `guinea pig` | `test subject` |
| `farm` (industry euphemism) | `factory farm` |

Full pattern list: [Open-Paws/no-animal-violence](https://github.com/Open-Paws/no-animal-violence)

### Security (three-adversary model)

Secret scanning via TruffleHog and the hardcoded-credentials pre-merge check guard against state surveillance, industry infiltration, and accidental data exposure. Sensitive repos (Tier 2/3) should additionally override `knowledge_base.opt_out: true` — see below.

### Test quality

Path instructions for test files enforce the mutation-testing mindset: every test must fail if the behavior it covers breaks. Snapshot traps and coverage theater are explicitly rejected.

### Instruction file integrity

The `.claude/**` path instruction scans for hidden Unicode characters — a known attack vector (Rules File Backdoor) where malicious instructions are embedded invisibly in CLAUDE.md or AGENTS.md files.

### Movement terminology

The tone instruction requires CodeRabbit to use movement-correct language in its review comments: `campaign`, `investigation`, `coalition`, `farmed animal`, `slaughterhouse`. It will not invent synonyms for these established domain terms.

---

## Tier-based overrides

Repos are classified by data sensitivity. See `ecosystem/repos.md` in the [context repo](https://github.com/Open-Paws/context) for current tier assignments.

| Tier | Examples | Recommended override |
|---|---|---|
| Tier 1 (public, no sensitive data) | platform, desloppify, graze-cli | Use this config as-is |
| Tier 2 (coalition or campaign data) | campaign tooling | `knowledge_base.opt_out: true` |
| Tier 3 (investigation data, witness info, legal) | investigation ops, gary | `knowledge_base.opt_out: true` |

### Minimal Tier 2/3 override

```yaml
# .coderabbit.yaml (repo root)
inheritance: true   # keep all other org defaults

knowledge_base:
  opt_out: true     # disable KB for this repo — investigation data must not train models
```

---

## CI workflows in this repo

This repo also ships two GitHub Actions workflows that can be deployed to target repos:

### `auto-merge.yml` — Wave 0 auto-merge gate

Merges `level-0` PRs automatically when all five gates pass:

1. PR is labeled `level-0` (no external state changes, agent-confirmed safe)
2. All required CI checks pass
3. Desloppify score meets repo threshold (Gary ≥80, platform ≥75, all others ≥70)
4. No-animal-violence scan has zero errors
5. A validation agent (gary, validation-agent, or stuckvgn) has left an APPROVE review

Hard safety constraints — auto-merge is **never** triggered when:
- Changes touch `.env`, secret, credential, or key files
- Changes modify `.github/workflows/` files
- PR carries a `level-1`, `level-2`, `level-3`, or `needs-human-review` label

If any gate fails, the workflow adds `needs-human-review` and posts a comment listing which gates failed.

### `no-animal-violence.yml` — speciesist language CI check

Runs `Open-Paws/no-animal-violence-action@v1` on every PR. Fails on errors; warnings do not block.

---

## Pre-commit hook

`.pre-commit-config.yaml` installs the no-animal-violence pre-commit hook for local development:

```yaml
repos:
  - repo: https://github.com/Open-Paws/no-animal-violence-pre-commit
    rev: v1.0.0
    hooks:
      - id: no-animal-violence
        files: \.(py|ts|js|md|yaml|yml)$
```

---

## Updating the org-wide config

1. Edit `.coderabbit.yaml` in this repo on a feature branch
2. Validate against the schema: `yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json`
3. Open a PR — the change affects every Open Paws repo that inherits this config, so treat it as a high-impact change
4. After merge, CodeRabbit picks up the new config within minutes; no deployment step required

The canonical source for the full config design is documented at:  
`open-paws-strategy/roadmap/coderabbit-central-config.yaml`  
Per-repo override configs: `open-paws-strategy/roadmap/coderabbit-repo-configs/`
