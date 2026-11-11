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
| `release-notes.yml` | Adds detail to the GitHub releases: under each entry, the **Release notes** section of the pull request's description and the issues it closes.                                 |

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

## Release notes

`release-notes.yml` has no trigger of its own: it runs as the last job of the repository's `release.yml`, after the job that runs semantic-release.

```yaml
release-notes:
  name: Release notes
  needs: release
  # A dry run creates no release to add notes to.
  if: ${{ !inputs.dry-run }}
  uses: modulane/workflows/.github/workflows/release-notes.yml@<sha> # vX.Y.Z
  permissions:
    contents: write # edit the GitHub releases
    issues: read # list the issues a pull request closes
    pull-requests: read # read the pull request descriptions
```

- **Where the notes come from.** The pull request template has a **Release notes** section. Under each entry of a release, the workflow adds the text of that section, then the issues the pull request closes. An empty section, or "None", adds no text.
- **Each release once.** A release it has handled gets a hidden `<!-- modulane/workflows release-notes -->` marker, and each run handles every release without one. The first run fills in the older releases too, and running the release workflow by hand fills in any release it missed. Editing a pull request after its release does not change that release: edit the release on GitHub instead.
- **Review the notes.** They are copied from the pull request description as written, so check that section before merging, as you would the code.

## Releases

Every merge to `main` runs semantic-release: a `feat:` title releases a minor version, a `fix:` title a patch, and a `!` a major one, which callers then have to adapt to. Changing a workflow's inputs, permissions or job names is breaking for its callers.

## License

[MIT](LICENSE)
