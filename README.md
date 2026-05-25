# Reproducer: dependabot/dependabot-core#14151

## Problem

This project sets `min-release-age=730` in `.npmrc`, telling npm to refuse
installing any package version published less than 730 days ago. Dependabot
ignores this setting and proposes updates to the latest version regardless
of how recently it was published.

## Expected behavior

Dependabot reads `min-release-age` from `.npmrc` and applies it as a
cooldown, proposing only versions that are at least 730 days old.

## Actual behavior

Dependabot opens a pull request updating `vue` to the latest version even
when that version was published within the 730-day window. Running
`npm install` after merging such a PR fails with:

```
npm error code ETARGET
npm error notarget No matching version found for vue@...
npm error notarget In most cases you or one of your dependencies are
npm error notarget requesting a package version that doesn't exist.
```

## How to reproduce

1. Fork this repository and enable Dependabot.
2. Dependabot opens a PR proposing `vue` be updated to the latest version.
3. Merge the PR and run `npm install`.
4. npm rejects the install because the proposed version is younger than
   the 730-day `min-release-age` threshold.

## Root cause

`UpdateChecker` in the `npm_and_yarn` ecosystem does not parse `.npmrc` for
`min-release-age` and does not convert it to a `ReleaseCooldownOptions`
value. The two systems are independent, creating two sources of truth that
can drift.

## Fix

See dependabot/dependabot-core#15132.
