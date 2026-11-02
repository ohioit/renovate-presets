# renovate-presets

Shared [Renovate](https://docs.renovatebot.com) configuration for repositories that deploy to Kubernetes. Extend it to get sensible defaults, wider detection of Kubernetes, Helm and Kustomize dependencies, and consistent commit messages.

## Quick start

Add this to your repo's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>ohioit/renovate-presets"]
}
```

That loads the [default preset](./default.json). [Application presets](#application-presets) are opt-in and are added separately.

## What you get

**Renovate's recommended setup**, with some settings to be aware of:

- Renovate's [`config:best-practices`](https://docs.renovatebot.com/presets-config/#configbest-practices): a dependency dashboard issue, pinned Docker and GitHub Action digests, weekly lock file maintenance, and a minimum release age for npm packages.
- Major updates for Docker images are enabled.
- **Renovate's PR rate limits are turned off**, so a repo with many outdated dependencies gets all its PRs at once. See [Overriding the defaults](#overriding-the-defaults) to set limits.

**Wider dependency detection:**

| Manager       | Also scans                                                                      |
| ------------- | ------------------------------------------------------------------------------- |
| `kubernetes`  | Every `.yaml` and `.yml` file (Renovate scans none by default)                  |
| `helm-values` | `helmrelease.yaml`, `helm-release.yaml` and `hr.yaml`, as well as `values.yaml` |

`kubernetes`, `helm-values`, `helmfile` and `kustomize` also read Jinja-templated versions of their files, such as `values.yaml.j2`.

It also picks up any `oci://` reference with a version in a YAML file, such as `oci://ghcr.io/stefanprodan/charts/podinfo:6.9.0`.

**Consistent commit messages**, which suit tools like release-please:

| Update                   | Commit type |
| ------------------------ | ----------- |
| Major                    | `feat!`     |
| Minor                    | `feat`      |
| Patch or digest          | `fix`       |
| Minor of a `0.x` package | `feat!`     |

GitHub Actions use `ci`, and mise tools and GitHub releases use `chore`, in place of `feat` or `fix`. Major updates keep the `!`. Because `0.x` minors can break at any time, they also go on a `major-` branch.

Major and `0.x` minor updates always use this format. The others need semantic commits turned on: Renovate detects this from your repo's history, or you can add `:semanticCommits` to `extends` to force it.

## Tracking other versions

For a version that no manager detects, add a comment directly above it, in any file:

```yaml
# renovate: datasource=github-releases depName=kubernetes-sigs/kustomize
kustomizeVersion: v5.4.0
```

```sh
# renovate: datasource=docker depName=ghcr.io/stefanprodan/podinfo
PODINFO_VERSION=6.9.0
```

The comment must end after `depName`. A line with anything after it, such as `versioning=semver`, is not picked up.

## Application presets

Add these only if you run the application. Each one is listed alongside the default preset:

```json
{
  "extends": [
    "github>ohioit/renovate-presets",
    "github>ohioit/renovate-presets//apps/kubectl.json5"
  ]
}
```

Include the `.json5` extension, because Renovate only tries `.json` by default.

| Preset                                                           | Use it if                              | What it does                                                 |
| ---------------------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------ |
| [`apps/kubectl.json5`](./apps/kubectl.json5)                     | You manage `kubectl` with mise         | Holds it to the Kubernetes minor version our clusters run    |
| [`apps/grafanaDashboards.json5`](./apps/grafanaDashboards.json5) | You deploy dashboards from grafana.com | Updates their revisions                                      |
| [`apps/ansibleRhel8.json5`](./apps/ansibleRhel8.json5)           | You use Ansible to manage RHEL 8 hosts | Holds Ansible tooling to the last versions that support them |

Each preset's file explains its details and limitations in comments.

## Pinning a version

Without a tag, you follow `main` and get changes as soon as they merge. To adopt changes on your own schedule, pin a [release](https://github.com/ohioit/renovate-presets/releases), and Renovate will open PRs when new releases come out. Each reference needs its own tag, otherwise it stays on `main`:

```json
{
  "extends": [
    "github>ohioit/renovate-presets#1.0.0",
    "github>ohioit/renovate-presets//apps/kubectl.json5#1.0.0"
  ]
}
```

Major releases change which updates you receive. The [changelog](./CHANGELOG.md) lists what changed in each one.

## Overriding the defaults

Beyond Renovate's recommended setup, these presets don't add grouping, automerge or schedule rules, so add yours to your repo's config, which takes priority over presets. For example, to restore rate limits and automerge minor and patch updates:

```json
{
  "extends": ["github>ohioit/renovate-presets"],
  "prConcurrentLimit": 10,
  "prHourlyLimit": 2,
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "matchCurrentVersion": "!/^v?0/",
      "automerge": true
    }
  ]
}
```

Keep the `matchCurrentVersion` line when automerging minors, so `0.x` updates that may be breaking still wait for review.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md).
