# NeuralGuard.Action

A GitHub Action that runs [NeuralGuard](https://github.com/rafaelarantes/NeuralGuard) against a pull request diff, posts the result as a comment and fails the check when Claude confirms a vulnerability.

## Usage

```yaml
name: NeuralGuard

on:
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: rafaelarantes/NeuralGuard.Action@v1
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

`fetch-depth: 0` is required, otherwise the base branch is not available to compute the diff.

## Inputs

| Input | Default | What it does |
|---|---|---|
| `anthropic-api-key` | required | Key used to ask Claude for a second opinion. |
| `claude-model` | `claude-opus-5` | Model that reviews the findings. Cheaper models cost less per pull request, stronger models dismiss more false positives. |
| `backend` | `codebert` | Local classifier that raises the findings, either `codebert` or `scratch`. |
| `threshold` | `0.6` | Confidence above which a finding is sent to Claude. |
| `block` | `true` | Whether a confirmed vulnerability fails the check. |
| `tool-version` | `1.0.0` | Version of the `NeuralGuard.Cli` tool to install. |
| `github-token` | `github.token` | Token used to post the comment. |

## What it costs

Claude is only called for snippets the local classifier already flagged above the threshold, so a pull request that adds no suspicious code costs nothing. Raise `threshold` to call Claude less often, lower it to catch more.

The pretrained model files are cached between runs, so only the first run on a repository pays the download.

## Making it a required check

Set `block: "true"` (the default), then add the workflow's job to the branch protection rules for your default branch. The check fails only when Claude confirms a finding, never on a false positive it dismissed.
