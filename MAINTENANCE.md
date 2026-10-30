# Maintenance

## Setup

```sh
mise install
```

This installs the pinned tools and the lefthook pre-commit hook, which formats staged files and
validates presets on every commit. Run the same checks CI runs with `mise run lint` and
`mise run validate`.

## Making a change

1. Branch from `main` and make your change.
2. Commit, then open a pull request with a [Conventional Commits](https://www.conventionalcommits.org)
   title.
3. Once CI passes, **squash merge**. The PR title becomes the commit message on `main`, so it is
   the title that counts; individual commit messages on the branch do not.

## Commit messages

```text
<type>(<optional scope>): <description>
```

| Type                                                  | Use for                                             | Release |
| ----------------------------------------------------- | --------------------------------------------------- | ------- |
| `fix`                                                 | a rule that behaved wrongly                         | patch   |
| `feat`                                                | a new preset or rule, or a widened rule             | minor   |
| `feat!` / `fix!`                                      | a change that alters what consumers get updates for | major   |
| `chore`, `ci`, `docs`, `refactor`, `build`, `deps`, … | everything else                                     | patch   |

Examples:

```text
fix(grafanaDashboards): match dashboards indented with tabs
feat(kubectl): allow 1.37
feat(default)!: drop the helm commit message override
```

A `!` (or a `BREAKING CHANGE:` footer) always means a major release. A message that is not a
conventional commit is left out of the changelog and does not count towards a release.

## Releasing

Releases are automated by [release-please](https://github.com/googleapis/release-please):

1. Every merge to `main` opens or updates a `chore(release): X.Y.Z` pull request. It bumps the
   version from the commits merged since the last release and writes `CHANGELOG.md`.
2. **Merge that pull request to release.** Merging it creates the `X.Y.Z` tag and GitHub release.

Until it is merged, nothing is released, so several changes can go out together. To force a
specific version, add a `Release-As: X.Y.Z` footer to a commit message.

## Upgrading kubectl after a cluster upgrade

Change the minor in `allowedVersions` in [`apps/kubectl.json5`](./apps/kubectl.json5), merge it as
`feat(kubectl): allow 1.NN`, then merge the release pull request. Repos that extend the preset
without a tag pick it up on their next Renovate run.
