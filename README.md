# Lintsure GitHub Action

Runs [Lintsure](https://lintsure.com) on every pull request. The check runs inside your own GitHub Actions runner and nothing is uploaded. The full report goes into the job summary.

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
          frameworks: gdpr,soc2   # optional
          fail-on: gap            # gap | partial | none
```

## Inputs

| Input | Default | What it does |
|---|---|---|
| `path` | `.` | Folder to check |
| `frameworks` | empty | `eu-ai-act`, `gdpr`, `soc2`, `iso-27001`, comma-separated. Empty uses `lintsure.config.yaml`, or all available. |
| `fail-on` | `gap` | `gap` fails on any Gap, `partial` on any Gap or Partial, `none` never fails |
| `ref` | `main` | Which version of [lintsure/lintsure](https://github.com/lintsure/lintsure) to build and run (branch, tag or commit). Pin a commit for repeatable results. |

## Output

`report`: the path to `lintsure-report.json`, for later steps.

## How it works

The action builds the Lintsure engine from source at `ref`, then runs the CLI on your code. It needs no npm package and no secrets. A self-test workflow in this repo checks that a planted problem fails the job and that every report carries the disclaimer.

## Help Lintsure grow

⭐ Star [lintsure/lintsure](https://github.com/lintsure/lintsure), share it with your team, [send a review](https://lintsure.com/review), or [request a demo](https://lintsure.com/demo) of the team platform.

Questions: [hello@lintsure.com](mailto:hello@lintsure.com) · Support: [support@lintsure.com](mailto:support@lintsure.com) · [Docs](https://lintsure.com/docs/github-action)

Apache-2.0.
