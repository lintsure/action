# Lintsure GitHub Action

Runs [Lintsure](https://lintsure.com) on every pull request. The check runs inside your own CI runner; nothing is uploaded. The report is added to the job summary.

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
          frameworks: gdpr,soc2   # optional, default: all
          fail-on: gap            # gap | partial | none
```

> Status: early. Needs the `lintsure` CLI on npm before it works.

A readiness tool, not an audit or legal advice. Apache-2.0.
