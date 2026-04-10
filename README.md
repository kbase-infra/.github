# Build Publish and Scan Workflow Templates
* [build_publish_scan.yaml](build_publish_scan.yaml)
* [two_step_build_publish_scan.yaml](two_step_build_publish_scan.yaml)

A GitHub Actions workflow template that you can install across this org, that will build a Docker image, push it to GitHub Container Registry (ghcr.io), and scan it for vulnerabilities with trivy.

## Installation
Click on "actions" -> new workflow -> choose the workflow

## Triggers

| Trigger | When |
|---|---|
| Push | On commits to `main`, `master`, or `develop` |
| Pull Request | On PRs targeting `main`, `master`, or `develop` |
| Release | When a GitHub Release is published |
| Manual | Via the Actions tab (`workflow_dispatch`) |

## What It Does

The workflow builds and pushes a Docker image to `ghcr.io/<owner>/<repo>`, with behavior that varies by trigger:

| Behavior | Push / PR / Manual | Release |
|---|---|---|
| Platforms built | `linux/amd64` only | `linux/amd64` + `linux/arm64` |
| Disk space cleanup | Only if manually toggled on | Always runs |
| SBOM generated | No | Yes |
| Trivy vulnerability scan | Yes — fails on CRITICAL | Skipped |

On non-release builds, a vulnerability report is written to the GitHub Actions job summary. The build fails if any CRITICAL CVEs are found.

## Runtime Input

When triggering manually via `workflow_dispatch`, one input is available:

| Input | Description | Default |
|---|---|---|
| `free_disk_space` | Frees disk space before the build (~1-2 min overhead). Useful for large images. | `false` |

## Customization

Since this is a template you own, the expected flow is to edit the file directly after installing it. Common things to adjust:

- **Branches** — update the branch list under `push` and `pull_request` to match your workflow
- **Platforms** — change the platform list if you always want multiplatform builds, or don't need arm64 at all
- **Failure threshold** — the build currently fails on `CRITICAL` vulnerabilities only; tighten this to also fail on `HIGH` if needed
- **Reported severities** — the Trivy scan reports `CRITICAL,HIGH,MEDIUM,LOW,UNKNOWN`; narrow this if you want less noise
- **Disk space cleanup** — currently auto-triggers on release builds; enable it always if your images are consistently large
- **SBOM generation** — currently tied to releases; move it earlier if you want SBOMs on every build

## Contributing

If you modify any workflow template file, you must update the version as part of your PR:

1. Bump the `#vYYYY.MM.DD` line at the top of the changed `.yaml` file
2. Update the version text in the corresponding `.properties.json` so it reflects in the Actions UI

This allows users to compare the version in their installed copy against the current one in this repo.

## Outputs

- A Docker image pushed to `ghcr.io/<owner>/<repo>` with auto-generated tags (branch name, PR number, semver on release)
- On releases: a multiplatform image and an SBOM attestation
- On non-releases: a Trivy vulnerability summary in the Actions job summary
