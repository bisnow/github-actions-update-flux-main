# Force-update flux-main branch

A composite GitHub Action that automatically updates the `flux-main` branch to match your current branch when specific conditions are met.

## When does it update?

The action updates `flux-main` when either:
- A container was built AND the image tag was successfully updated
- No container was built AND Kubernetes files changed

## Usage

This action is designed to be used after other jobs in your workflow. It requires outputs from previous jobs to determine whether to update the `flux-main` branch.

### Basic Example

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      build-container: ${{ steps.check-build.outputs.build }}
      k8s-changed: ${{ steps.check-k8s.outputs.changed }}
    steps:
      # Your build steps here
      - id: check-build
        run: echo "build=true" >> $GITHUB_OUTPUT
      - id: check-k8s
        run: echo "changed=false" >> $GITHUB_OUTPUT

  update-image-tag:
    needs: build
    runs-on: ubuntu-latest
    if: needs.build.outputs.build-container == 'true'
    steps:
      # Your image tag update steps here
      - run: echo "Updating image tag..."

  update-flux-main:
    needs: [build, update-image-tag]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - uses: bisnow/github-actions-update-flux-main@v1
        with:
          build-container: ${{ needs.build.outputs.build-container }}
          k8s-changed: ${{ needs.build.outputs.k8s-changed }}
          update-image-tag-result: ${{ needs.update-image-tag.result }}
```

### Example with custom branch

```yaml
  update-flux-main:
    needs: [build, update-image-tag]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - uses: bisnow/github-actions-update-flux-main@v1
        with:
          build-container: ${{ needs.build.outputs.build-container }}
          k8s-changed: ${{ needs.build.outputs.k8s-changed }}
          update-image-tag-result: ${{ needs.update-image-tag.result }}
          branch-name: 'my-custom-branch'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `build-container` | Whether a container was built (`true`/`false`) | Yes | - |
| `k8s-changed` | Whether Kubernetes files changed (`true`/`false`) | Yes | - |
| `update-image-tag-result` | Result of the update-image-tag job (`success`/`failure`/`skipped`) | Yes | - |
| `branch-name` | Branch name to push to flux-main | No | Current branch (`github.ref_name`) |

## Important Notes

- This action uses `--force` to push to `flux-main`, so the branch will be overwritten
- The `update-flux-main` job should use `if: always()` to run even if previous jobs fail
- Use `needs.job-name.result` to pass the job result status
