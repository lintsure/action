# Security policy

## Reporting a vulnerability

Please email **[support@lintsure.com](mailto:support@lintsure.com)** with "Security" in the subject. Include what you found, how to reproduce it, and the version or commit.

Please don't open a public GitHub issue for a security problem. We'll reply to confirm we received it and keep you updated until it's resolved.

## Supported versions

Fixes go into the latest release and the `v1` tag. Older versions are not patched.

## What this action does with your code

It reads the files in your runner and makes no network calls with them. It downloads the Lintsure engine's source from GitHub and builds it. Only the opt-in pull request comment talks to GitHub's API, with your own workflow token, and it never includes code.
