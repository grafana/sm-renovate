# Allowing Renovate to automerge PRs

## Introduction

As a team, SM has acknowledge the need of automatically merging an agreed subset of renovate PRs. Renovate allows to define, in its config, a series of rules that govern whether it should attempt to merge PRs for certain packages, dependency types, and ecosystems by itself, instead of relying on a human to perform the merge. As this mechanism is built into renovate, and can be defined and reviewed as code, it seems the obvious choice for deciding _what_ will be subject to be merged automatically.

The goal of this document is to agree on the means to allow renovate to perform this merge action, which as of today is not possible even if we configure it to do so. Deciding _what_ dependencies are subject to be automerged is not the scope of this document.

## Problem

Renovate already has a comprehensive system of rules that can be leveraged to specify what packages and under which circumstances should be merged automatically. However, for that to work, renovate has to be able to actually perform the merge.

At the point of writing, SM repositories are configured to enforce [a series of requirements](https://github.com/grafana/deployment_tools/blob/1c48dc8e28c43e990274882c7c090ba4bd7c9950/terraform/ancillary/github/synthetic-monitoring/repos.tf#L136) for a PR to be allowed to be merged into the default branch. These requirements include things such as linear history, commit message conventions, passing CI/CD checks, etc. However, the one requirement that poses a problem to automerging PRs, is [the one requiring at least one approval](https://github.com/grafana/deployment_tools/blob/1c48dc8e28c43e990274882c7c090ba4bd7c9950/terraform/ancillary/github/synthetic-monitoring/repos.tf#L166) from members with write access. This requirement means that even if Renovate is configured to try and merge a PR automatically, GitHub will prevent it from doing so unless the PR has been approved.

The goal of this document is to gather consensus on how we allow renovate to merge PRs without human approval, while at the same time:
- Keep the approval requirement for actors other than Renovate
- Keep all the other requirements for every actor, renovate included

## Proposal 1: Extended privileges for `grafanarenovatebot`

In our repositories, Renovate runs as a scheduled workflow in GHA, and acts as the [`grafanarenovatebot` app](https://github.com/apps/grafanarenovatebot). A solution to the automerge problem could be to allow this user specifically to bypass this requirement.

This should be doable by adding a "Bypass" entry for this user in the ruleset that requires approvals, under Rules > Rulesets > [Add Bypasss] button.

It should be noted that:
- The ruleset that requires CI/CD to pass can be a different one, without any bypass.
- Renovate itself will not attempt to automerge a PR if CI/CD does not pass, or if there is no CI/CD (unless [`ignoreTests`](https://docs.renovatebot.com/configuration-options/#ignoretests) is set, which we shouldn't).

➕ Pros:
- Simple: Only a config change, no coding required.

➖ Cons:
- This change needs to be either brought into terraform, or documented somewhere as a requirement.

## Proposal 2: Approver bot

Instead of granting the renovate user bypass privileges, we could have a separate bot granting approvals to renovate PRs, so Renovate can automerge them just as if a human had approved the PR.

This is an approach similar to the [`dependabot-automerge` workflow](https://github.com/grafana/security-github-actions/blob/main/.github/workflows/dependabot-automerge.yaml) in `grafana/security-github-actions`, but it should be noted that we probably don't want to use this particular workflow for a number of reasons.

➕ Pros:
- No need to create bypass permissions, or make them consistent across repositories.

➖ Cons:
- We'd need another application installed company-wide.
- This application needs to be given write permissions for its approvals to be valid.
- We'd need to write the code for the approver scheduled workflow.
- We'd need to propagate the approver workflow to all our repositories.
