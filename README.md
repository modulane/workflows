# workflows

Reusable GitHub Actions workflows for modulane repositories. Each repository calls them instead of keeping its own copy, so a change made here reaches every repository at once.

## What it holds

| Workflow            | What it does                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pr-title.yml`      | Checks that the pull request title is a lowercase Conventional Commit. With `package-scope: true`, a monorepo's `feat:` or `fix:` title must also name the package it releases. |
| `pr-labels.yml`     | Labels each pull request from its title (kind and `release:` label) and assigns its author, unless the author is a bot.                                                         |
| `ai-disclosure.yml` | Fails until the pull request's **AI use** section says whether AI tools were used, and how, with [modulane/ai-disclosure](https://github.com/modulane/ai-disclosure).           |
| `labels.yml`        | Syncs the repository labels from its `.github/labels.json`.                                                                                                                     |
| `zizmor.yml`        | Audits the security of the repository's workflows.                                                                                                                              |

`ci.yml` and `release.yml` stay in each repository: CI runs the repository's own checks, and the release differs between a package, an action and a monorepo. This repository runs its workflows on its own pull requests too.

## Usage

Each repository keeps a short workflow per check, with its own trigger, that calls the workflow here. [repo-template](https://github.com/modulane/repo-template) holds the full set, and new repositories start from it. For example:

```yaml
name: Pull request title

on:
  pull_request:
    types: [opened, edited, reopened, synchronize]

permissions: {}

jobs:
  pr-title:
    name: Pull request title
    uses: modulane/workflows/.github/workflows/pr-title.yml@<sha> # vX.Y.Z
    permissions:
      contents: read # list the files of a pull request in a private repository
      pull-requests: read # read the pull request title and its files
```

- **Pinned to a commit.** Calls are pinned to the commit SHA of a release, like every other action, and Dependabot proposes the new release in each repository.
- **Permissions.** A called workflow gets at most the permissions its caller grants, so each call grants what the workflow's jobs ask for.
- **Check names.** A check from a called workflow is named after the calling job and the called one, for example **Pull request title / Conventional Commit title**. The main ruleset requires the checks by these names.
- **No concurrency in the caller.** The workflows here set their own concurrency group; the same group in the caller would cancel the call.

## Releases

Every merge to `main` runs semantic-release: a `feat:` title releases a minor version, a `fix:` title a patch, and a `!` a major one, which callers then have to adapt to. Changing a workflow's inputs, permissions or job names is breaking for its callers.

## License

[MIT](LICENSE)
