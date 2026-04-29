# Codex Issue Labeler AWI PoC

This repository is a minimal proof-of-concept for the `issue-labeler.yml` Agentic Workflow Injection pattern seen in Codex-based GitHub Actions workflows.

It mirrors the vulnerable pattern:

- untrusted issue title/body is interpolated directly into the prompt for `openai/codex-action`
- the model output is treated as trusted label-selection output
- a downstream job with `issues: write` applies returned labels with `gh issue edit`

## Scope

This PoC is intended for controlled security validation only.

- It does not claim exploitation against any upstream repository.
- It reproduces the workflow pattern in an isolated test repository.
- To execute the workflow end-to-end, add an Actions secret named `CODEX_OPENAI_API_KEY`.

## Files

- `.github/workflows/issue-labeler.yml`: minimal vulnerable workflow
- `poc/payload.md`: sample attacker-controlled issue content

## Setup

1. Add the repository Actions secret `CODEX_OPENAI_API_KEY`.
2. Ensure the repository has labels such as `bug`, `github-action`, `auth`, and `codex-label`.
3. Open a new issue using the sample text from `poc/payload.md`.

## Expected Behavior In The PoC

If the model follows the injected instruction instead of the intended labeling rules, the `apply-labels` job may add attacker-chosen labels to the issue.

That demonstrates the core risk chain:

`untrusted issue content -> model prompt -> model output -> issue label write`
