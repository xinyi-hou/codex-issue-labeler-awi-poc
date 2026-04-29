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

- `.github/workflows/issue-labeler.yml`: minimal vulnerable workflow using `openai/codex-action`
- `.github/workflows/simulated-obedient-labeler.yml`: deterministic simulation of the same prompt-to-label-write chain without an external model dependency
- `poc/payload.md`: sample attacker-controlled issue content

## Setup

1. Ensure the repository has labels such as `bug`, `github-action`, `auth`, and `codex-label`.
2. Open a new issue using the sample text from `poc/payload.md`.
3. For the real `openai/codex-action` path, add the repository Actions secret `CODEX_OPENAI_API_KEY`.

## Expected Behavior In The PoC

If the model follows the injected instruction instead of the intended labeling rules, the `apply-labels` job may add attacker-chosen labels to the issue.

The repository also includes a deterministic simulation workflow so the downstream label-write behavior can be validated in a controlled way even without an external model API key.

That demonstrates the core risk chain:

`untrusted issue content -> model prompt -> model output -> issue label write`

## Observed Controlled Validation

The deterministic simulation workflow has already been exercised in this repository:

- Demonstration issue: `#2`
  - https://github.com/xinyi-hou/codex-issue-labeler-awi-poc/issues/2
- Successful simulation run:
  - https://github.com/xinyi-hou/codex-issue-labeler-awi-poc/actions/runs/25087480228

In that run, attacker-controlled issue content containing `{"labels":["bug","github-action","auth"]}` caused the workflow to apply the labels `bug`, `github-action`, and `auth` to the issue.
