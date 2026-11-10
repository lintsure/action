# Lintsure GitHub Action

Runs [Lintsure](https://lintsure.com) on every pull request. The check runs inside your own GitHub Actions runner and Lintsure sends your code nowhere. Findings show on the changed lines, and the full report goes into the job summary.

> **Lintsure is a readiness tool, not an audit, a certification or legal advice.** It checks code only, against our reading of the publicly available text of each framework. Work outside code still has to be done, and a qualified person should review the results. See the [disclaimer](https://github.com/lintsure/lintsure/blob/main/DISCLAIMER.md).

## Use it

```yaml
name: lintsure
on: [pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: lintsure/action@v1
        with:
          frameworks: gdpr,owasp-top10   # optional
          fail-on: gap            # gap | partial | none
```

## Inputs

| Input | Default | What it does |
|---|---|---|
| `path` | `.` | Folder to check |
| `frameworks` | empty | `eu-ai-act`, `gdpr`, `owasp-top10`, `owasp-llm`, `cwe-top25`, `cra`, comma-separated. Empty uses `lintsure.config.yaml`, or all available. |
| `fail-on` | `gap` | `gap` fails on any Gap, `partial` on any Gap or Partial, `none` never fails |
| `new-only` | `true` | When `.lintsure/baseline.json` exists, fail only on findings that are new since the baseline. Existing code doesn't block your first pull request. |
| `annotations` | `true` | Show each open finding on its line in the pull request. The runner prints these; no extra permissions. |
| `sarif` | `false` | Also write `lintsure.sarif`, for GitHub code scanning. |
| `comment` | `false` | Post one summary comment on the pull request and update it on later runs. See below. |
| `ref` | `main` | Which version of [lintsure/lintsure](https://github.com/lintsure/lintsure) to build and run (branch, tag or commit). Pin a commit for repeatable results. |

## Outputs

- `report`: the path to `lintsure-report.json`
- `sarif`: the path to `lintsure.sarif`, when `sarif` is true

## Baseline and accepted findings

Run `npx lintsure baseline` once and commit `.lintsure/baseline.json`. After that, each pull request shows what is new, fixed or unchanged, and with `new-only` only new findings fail the job.

To accept a finding after review, add it to `.lintsure/accepted.yaml` with its fingerprint (from the JSON report), your reason, your name, the date and an optional review date. Accepted findings don't fail the job; past the review date they count as open again.

Both files contain rule ids, file paths, one-way fingerprints and people's reasons. They never contain code.

## The pull request comment (opt-in)

With `comment: true`, the action posts one comment on the pull request and edits it on later runs. It lists new findings with their file and line, the finding and the proposed change. **This sends that text to GitHub through your own workflow token. It never includes code.** Give the job permission to write to pull requests:

```yaml
permissions:
  contents: read
  pull-requests: write
```

## Code scanning (optional)

```yaml
permissions:
  contents: read
  security-events: write
steps:
  - uses: actions/checkout@v4
  - id: lintsure
    uses: lintsure/action@v1
    with: { sarif: true, fail-on: none }
  - uses: github/codeql-action/upload-sarif@v3
    with: { sarif_file: lintsure.sarif }
```

## How it works

The action builds the Lintsure engine from source at `ref`, then runs the CLI on your code. It needs no npm package and no secrets, and only the opt-in comment talks to GitHub's API. A self-test workflow in this repo checks that a planted problem fails the job and that every report carries the disclaimer.

## Help Lintsure grow

⭐ Star [lintsure/lintsure](https://github.com/lintsure/lintsure), share it with your team, [send a review](https://lintsure.com/review), or [request a demo](https://lintsure.com/demo) of the team platform.

Questions: [hello@lintsure.com](mailto:hello@lintsure.com) · Support: [support@lintsure.com](mailto:support@lintsure.com) · [Docs](https://lintsure.com/docs/github-action)

Apache-2.0.
