# Lintsure GitHub Action

[![Version](https://img.shields.io/github/v/release/lintsure/action?label=version&color=6289FE)](https://github.com/lintsure/action/releases)
[![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Lintsure-141413?logo=github)](https://github.com/marketplace/actions/lintsure)
[![Self-test](https://github.com/lintsure/action/actions/workflows/self-test.yml/badge.svg)](https://github.com/lintsure/action/actions/workflows/self-test.yml)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-141413)](LICENSE)
[![Engine on npm](https://img.shields.io/npm/v/lintsure?label=engine&color=141413)](https://www.npmjs.com/package/lintsure)
[![Listed by OWASP](https://img.shields.io/badge/listed%20by-OWASP-0B6E4F)](https://owasp.org/www-community/Source_Code_Analysis_Tools)
[![Docs](https://img.shields.io/badge/docs-lintsure.com-0B6E4F)](https://lintsure.com/docs/github-action)

**Check the code your AI coding agents write, on every pull request.**

Agents write code fast, and some of it carries problems a quick review misses: a key left in a file, a database anyone can read, an email address written to the logs. This action runs [Lintsure](https://lintsure.com) on each pull request and points at the exact line. It runs inside your own GitHub Actions runner and sends your code nowhere.

## What you get

- **Findings on the changed lines.** Each one shows on its line in the pull request, with what is wrong and a proposed fix. The full report goes into the job summary.
- **114 rules** for the problems AI-written code often has: secrets in source, open databases, personal data in logs, unsafe agent settings, risky skills and tool servers.
- **Eight frameworks.** Each finding is mapped to the controls it touches in the OWASP Top 10, the OWASP API Security Top 10, the OWASP LLM Top 10, the OWASP Agentic Skills Top 10, the CWE Top 25, GDPR, the EU AI Act and the EU Cyber Resilience Act.
- **A gate you control.** Fail the job on a clear problem, on anything unsure, or never. Old findings don't block your first pull request.
- **The same answer every time.** The check is a fixed rule engine, not a language model. Same code in, same result out.
- **No secrets and no account.** Nothing to sign up for, and nothing leaves the runner unless you turn on the pull request comment.

> **Lintsure is a readiness tool, not an audit, a certification or legal advice.** It checks code only, against our reading of the publicly available text of each framework. Work outside code still has to be done, and a qualified person should review the results. See the [disclaimer](https://github.com/lintsure/lintsure/blob/main/DISCLAIMER.md).

## Use it

Add this file as `.github/workflows/lintsure.yml`:

```yaml
name: lintsure
on: [pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: lintsure/action@v1
```

That is the whole setup. To choose frameworks or change when the job fails:

```yaml
      - uses: lintsure/action@v1
        with:
          frameworks: gdpr,owasp-top10   # optional
          fail-on: gap            # gap | partial | none
```

## Start without blocking anyone

Two settings let a team switch it on safely:

- `observe: true` reports when a run would have failed and lets it pass. Watch it for a week, then turn it off.
- A baseline, made once with `npx lintsure baseline`, means only findings that are new since then can fail a job.

## Other ways to use Lintsure

This action is the pull request check. The same engine also runs earlier, while the code is being written:

| Way | What you run | What it does |
|---|---|---|
| CLI | `npx lintsure scan` | A report in your terminal, when you ask |
| MCP server | `npx lintsure setup` | Your agent can check its own work and explain a finding |
| Plugin | `npx lintsure plugin` | Guards that run on each edit and command, for Claude Code, Cursor, Codex and Gemini CLI |

More in [lintsure/lintsure](https://github.com/lintsure/lintsure) and [lintsure/plugin](https://github.com/lintsure/plugin).

## Inputs

| Input | Default | What it does |
|---|---|---|
| `path` | `.` | Folder to check |
| `frameworks` | empty | `eu-ai-act`, `gdpr`, `owasp-top10`, `owasp-api`, `owasp-llm`, `owasp-ast`, `cwe-top25`, `cra`, comma-separated. Empty uses `lintsure.config.yaml`, or all available. |
| `fail-on` | `gap` | `gap` fails on any Gap, `partial` on any Gap or Partial, `none` never fails |
| `preset` | empty | `security`, `security-privacy` or `all`. Used when `frameworks` is empty and there is no `lintsure.config.yaml`. |
| `changed-only` | `auto` | On `pull_request` events, list only findings on lines the pull request changed. The action fetches the base branch for this. |
| `show-all` | `false` | Also list medium-confidence findings, which are held back by default. |
| `observe` | `false` | With `fail-on`: report when the run would have failed, and pass anyway. Use it to watch the check for a while before it blocks pull requests. |
| `new-only` | `true` | When `.lintsure/baseline.json` exists, fail only on findings that are new since the baseline. Existing code doesn't block your first pull request. |
| `annotations` | `true` | Show each open finding on its line in the pull request. The runner prints these; no extra permissions. |
| `sarif` | `false` | Also write `lintsure.sarif`, for GitHub code scanning. |
| `comment` | `false` | Post one summary comment on the pull request and update it on later runs. See below. |
| `ref` | `v0.3.16` | Which version of [lintsure/lintsure](https://github.com/lintsure/lintsure) to build and run (branch, tag or commit). Defaults to the latest release tag; use `main` for unreleased changes, or a commit for fully repeatable results. |

Each rule is high or medium confidence, set from findings reviewed one by one against the code on public projects. See [the field test](https://lintsure.com/field-test). By default only high-confidence findings are listed and the rest are counted. Set `show-all: true` to list them all. Every report ends with a Not covered section that says what the run didn't check.

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

SARIF results carry a level by risk (high `error`, medium `warning`, low `note`) and a `security-severity`, so code scanning can rank them.

## How it works

The action builds the Lintsure engine from source at `ref`, then runs the CLI on your code. It needs no npm package and no secrets, and only the opt-in comment talks to GitHub's API. A self-test workflow in this repo checks that a planted problem fails the job and that every report carries the disclaimer.

## Help Lintsure grow

⭐ Star [lintsure/lintsure](https://github.com/lintsure/lintsure), share it with your team, [send a review](https://lintsure.com/review), or [request a demo](https://lintsure.com/demo) of the team platform.

Questions: [hello@lintsure.com](mailto:hello@lintsure.com) · Support: [support@lintsure.com](mailto:support@lintsure.com) · [Docs](https://lintsure.com/docs/github-action)

Apache-2.0.
