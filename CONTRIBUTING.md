# Contributing

How to work on this repo: local setup, commit conventions, CI and releases. To use the presets,
see the [README](./README.md).

## Setup

Install [mise](https://mise.jdx.dev/getting-started.html), then from the repo root:

```sh
mise trust
mise install
```

This installs the pinned tools and a lefthook pre-commit hook. The hook runs on the files you
stage: it formats JSON, JSON5, YAML and Markdown, lints workflows and shell scripts, validates
`.json5` presets, and refreshes the mise lockfile if you changed the mise config.

Run the same checks as CI at any time:

| Command             | Checks                                                 |
| ------------------- | ------------------------------------------------------ |
| `mise run lint`     | Formatting, shell scripts and GitHub Actions workflows |
| `mise run validate` | `default.json`, `apps/*.json5` and `.renovaterc.json5` |

The hook does not validate `default.json`, so run `mise run validate` after editing it.

## Making a change

1. Branch from `main` and make your change.
2. Open a pull request with a [Conventional Commits](https://www.conventionalcommits.org) title.
3. Get one approval and a passing `Build Success` check. Pushing new commits dismisses earlier
   approvals, and the last push must be approved by someone other than its author.
4. **Squash merge**, and check the commit title in the merge dialog before confirming.

The squashed commit title is what drives the changelog and version bump, and no CI check enforces
its format. GitHub suggests the PR title, except on a single-commit PR, where it suggests that
commit's title instead. Rebase merging is also enabled, but it would put every branch commit into
the changelog, so avoid it.

**Merging takes effect immediately for consumers who don't pin a tag**, because they read presets
from `main`. Waiting for a release only holds a change back from consumers who pin.

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

## Editing presets

- Keep presets flat and self-contained. A tag only applies to the reference it is written on, so a
  preset that referenced another file here could not pass its tag along.
- Rules that apply to everyone go in `default.json`. So do rules for a particular tool, as long as
  they change nothing in repos without that tool and need no local decision, so that consumers get
  them without having to know they need them. Anything else, such as a version hold that depends on
  what you run, goes in a new or existing `apps/<name>.json5`, which consumers opt in to.
- Give new rules a `description`, and in `.json5` files add a comment explaining anything
  non-obvious, such as why a version is held.
- When you add an `apps/` preset, add it to the README table and describe its usage and limitations
  in comments in the preset itself. The README stays a summary.

### Upgrading kubectl after a cluster upgrade

Change the minor in `allowedVersions` in [`apps/kubectl.json5`](./apps/kubectl.json5) and merge it
as `feat(kubectl): allow 1.NN`. Then merge the release pull request so that consumers who pin a tag
can move to it.

## Changing tools

Tools are declared in [`.mise/config.toml`](./.mise/config.toml).

- Name the backend explicitly, such as `"aqua:rhysd/actionlint"` or `"npm:oxfmt"`. `node` is the
  exception.
- Commit the updated `.mise/mise.lock` and `.mise/locks/`. The hook regenerates them, and CI installs
  with `--locked`, so it fails if they are out of date.
- Don't reformat `.mise/locks/`: the files are digest-pinned in `mise.lock`.
- If a CI job needs the new tool, add it to that job's `tools` list in
  [`ci.yaml`](./.github/workflows/ci.yaml).

## Automation

| Workflow                                                  | Runs on                  | Does                                                           |
| --------------------------------------------------------- | ------------------------ | -------------------------------------------------------------- |
| [CI](./.github/workflows/ci.yaml)                         | Pull requests and `main` | Runs each lint and validation task; reports `Build Success`    |
| [Release Please](./.github/workflows/release-please.yaml) | Pushes to `main`         | Maintains the release pull request and creates releases        |
| [Stale](./.github/workflows/stale.yaml)                   | Daily                    | Marks items stale after 180 days and closes them 14 days later |

- `Build Success` is the required check. It passes when the lint jobs are skipped, which is what
  happens on release pull requests.
- Stale ignores anything labelled `renovate`.
- Release Please and Stale authenticate as a GitHub App, using the `RELEASE_BOT_CLIENT_ID` and
  `RELEASE_BOT_PRIVATE_KEY` repository secrets.
- Renovate also runs on this repo, configured by [`.renovaterc.json5`](./.renovaterc.json5), which
  extends this repo's own `default.json`. Its pull requests follow the same merge process.

## Releasing

Releases are automated by [release-please](https://github.com/googleapis/release-please):

1. Every merge to `main` opens or updates a `chore(release): X.Y.Z` pull request. It bumps the
   version from the commits merged since the last release and writes `CHANGELOG.md`.
2. **Merge that pull request to release.** Merging it creates the `X.Y.Z` tag and GitHub release.

Several changes can go out together by leaving it open. To force a specific version, add a
`Release-As: X.Y.Z` footer to a commit message.
