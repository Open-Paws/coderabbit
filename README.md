# Open Paws — CodeRabbit Central Config

This repo contains the org-wide default CodeRabbit configuration for all Open Paws repositories.

CodeRabbit reads `.coderabbit.yaml` from this repo automatically for any repo that does not have its own config file.

## How it works

- Repos **without** a `.coderabbit.yaml`: use this config
- Repos **with** their own `.coderabbit.yaml`: fully override this config (or merge if `inheritance: true`)

## Key settings

- `request_changes_workflow: true` — required for auto-merge gate checks
- `knowledge_base.opt_out: false` with `learnings.scope: local` — enables CLAUDE.md reading, keeps learnings repo-scoped
- Semgrep NAV integration — no-animal-violence scan runs inside every review
- Secret scanning (betterleaks + TruffleHog) on every PR
- Pre-merge checks: no hardcoded credentials, no speciesist idioms

## Sensitive repos (Tier 2/3)

Investigation data, witness info, and autonomous agent repos override with `knowledge_base.opt_out: true`.
See `ecosystem/repos.md` in the strategy repo for tier classifications.

## Source

Config maintained at: `open-paws-strategy/roadmap/coderabbit-central-config.yaml`
Per-repo override configs: `open-paws-strategy/roadmap/coderabbit-repo-configs/`
