# `renovate-approve`

This action allows to automatically approve PRs coming from Grafana's self hosted renovate bot.

## Example usage

> [!WARNING]
> **Always** gate this workflow behind an `if` statement as shown below, and run it **only** `on.pull_request`.

The action has built-in logic similar to the `if` statement, but it is always recommended from a security perspective to not just run the job altogether for PRs not coming from renovate.

```yaml
name: Approve renovate PRs

on:
  pull_request:

jobs:
  approve:
    permissions:
      # Needed for logging into vault.
      contents: read
      id-token: write
    runs-on: ubuntu-latest
    # Do not bother running if PR is not authored by grafanarenovatebot.
    # https://woodruffw.github.io/zizmor/audits/#bot-conditions
    if: github.event.pull_request.user.login == 'grafanarenovatebot[bot]' && github.repository == github.event.pull_request.head.repo.full_name
    steps:
      - uses: grafana/sm-renovate/actions/renovate-approve
```
