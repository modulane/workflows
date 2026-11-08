# workflows

Reusable GitHub Actions workflows for modulane repositories. Each repository calls them instead of keeping its own copy, so a change made here reaches every repository at once.

> **Status:** planned.

## What it will hold

- **CI passed:** the single job that the main ruleset requires, gathering the repository's own checks.
- **Pull request title:** a lowercase Conventional Commit, with the package scope check for monorepos.
- **Pull request labels:** the kind and `release:` labels from the title, and the author assigned.
- **AI disclosure**, **labels sync** and **zizmor**.

## Usage

```yaml
jobs:
  pr-title:
    uses: modulane/workflows/.github/workflows/pr-title.yml@<sha> # vX.Y.Z
```

Calls are pinned to a commit SHA, like every other action, and Dependabot keeps them up to date.

[repo-template](https://github.com/modulane/repo-template) stays the reference setup: it holds the calls to these workflows, and new repositories still start from it.

## License

[MIT](LICENSE)
