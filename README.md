# update-image-tag-action

Github Action to update image tag in kustomiztion.yaml

## Example Usage

```yaml
name: Deployment

on:
  push:
    branches:
      - main

jobs:
  build_deploy_image:
    runs-on: ubuntu-latest
    outputs:
      build-sha: ${{ github.sha }}
    steps:
      - uses: actions/checkout@v3
      - uses: docker/login-action@v2
        with:
          registry: gcr.io
          username: _json_key
          password: ${{ secrets.YOUR_GCR_JSON_KEY }}
      - uses: docker/setup-buildx-action@v2
      - uses: docker/build-push-action@v3
        with:
          push: true
          tags: gcr.io/iamagcpproject/imanimagename:${{ github.sha }}

  create_update_tag_pr_staging:
    runs-on: ubuntu-latest
    needs: build_deploy_image
    steps:
      - uses: actions/checkout@v3
      - uses: tibdex/github-app-token@v1
        id: generate-token
        with:
          app_id: ${{ secrets.YOUR_GITHUB_APP_ID }}
          private_key: ${{ secrets.YOUR_GITHUB_APP_PRIVATE_KEY }}
      - uses: jackpocket/update-image-tag-action@v1
        with:
          token: ${{ steps.generate-token.outputs.token }}
          branch: staging
          repo: organization/reponame
          tag: ${{ needs.build.outputs.build-sha }}
          path: k8s/path/to/folder # the directory path that contains the kustomiztion.yaml file
          image-name: gcr.io/iamagcpproject/imanimagename
          source-repo: ${{ github.repository }}
```
