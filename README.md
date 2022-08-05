# update-image-tag-action
Github Action to update image tag in kustomiztion.yaml

## Example Usage

```yaml
name: build-image

on:
  push:
    branches: main

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      build-sha: ${{ github.sha }}
    steps:
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - id: 'auth'
      name: Login to GCR
      uses: docker/login-action@v1
      with:
        registry: gcr.io
        username: _json_key
        password: ${{ secrets.GCR_JSON_KEY }}

    - name: Build and push
      uses: docker/build-push-action@v2
      with:
        push: true
        tags: gcr.io/iamagcpproject/imanimagename:${{ github.sha }}
        
  deployment:
    environment: staging
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy
      uses: jackpocket/update-image-tag-action@latest
      with:
        token: ${{ secrets.GH_PAT }}
        branch: staging
        repo: organization/reponame
        tag: ${{ needs.build.outputs.build-sha }}
        path: k8s/path/to/folder # the directory path that contains the kustomiztion.yaml file
        image-name: gcr.io/iamagcpproject/imanimagename
        source-repo: ${{ github.repository }}
```
