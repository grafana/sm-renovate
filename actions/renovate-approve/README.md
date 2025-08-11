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

## Working around `CODEOWNERS` requirement with `force-merge`

GitHub apps cannot be CODEOWNERS ([open issue](https://github.com/orgs/community/discussions/23064)). This means that if your repository has the 

```
[ ] Require review from Code Owners
```

check enabled (typically found in `/settings/rules/`), then this action's approval will be inconsequential and the PR won't be automatically merged.

To work around this GitHub limitation, you can instruct this action to not only approve, but also forcefully merge the PR, bypassing the CODEOWNERS requirement. To do so, call the action with the following properties:

> [!NOTE]
> To be able to forcefully merge PRs, you'll need to add `sm-approver-app` to the Bypass list of your ruleset.

> [!WARNING]
> If `force-merge` is used, other CI/CD jobs will not be waited for. You'll want to manually add `needs` entries to the merge job.

```yaml
name: Merge renovate PRs

on:
  pull_request:

jobs:
  merge:
    needs:
      - other-checks
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
        with:
          require-auto-merge: true # Neither approve nor merge PRs where auto-merge was not requested.
          force-merge: true # Forcefully merge (bypassing checks) after approval.
```

