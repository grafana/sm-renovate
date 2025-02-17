# Automerge workflow for renovate PRs

## Introduction

As a team, SM has acknowledge the needs and benefits of opt-in automerge for renovate PRs. This means that we want to define, in the renovate config, a series of rules that govern whether we certain packages, dependency types, and ecosystems are allowed to be merged without a PR, or without human approval.

Renovate already offers a mechanism for this, and deciding what will be automerged is not the scope of this document. How we make automerge happen at the repository policy level, however, is.

## Problem

Renovate already has a comprehensive system of rules that can be leveraged to specify what packages and under which circumstances should be merged automatically. However, for that to work, renovate has to be able to actually perform the merge.

This is currently not possible, as all SM repositories are configured to require at last one approval from members with write access (and potentially, `CODEOWNERS`) in order for a PR to be merged into main. This means that even if Renovate is configured to try and merge a PR automatically, GitHub will prevent it from doing so.

The goal of this document is to gather consensus on how we make this possible.

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
