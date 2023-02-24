# August Ash Shared Workflows

[![GitHub Release](https://img.shields.io/github/tag/augustash/workflows.svg?logo=github&style=for-the-badge)](https://github.com/augustash/workflows/tags)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

This repository is for sharing August Ash organization GitHub Actions workflows.

## ⚙️ Usage

The following is a simple example of using a workflow for manual deployment.

```yaml
# File: .github/workflows/workflow.yml
name: "Manual deployment"
run-name: "Manual deployment to ${{ inputs.deploy_target }} by @${{ github.actor }} 🚀"

on:
  workflow_dispatch:
    inputs:
      deploy-environment:
        description: "GitHub environment to run against"
        required: true
        type: environment
      deploy-target:
        description: "The deployment target environment"
        options:
          - "staging"
          - "production"
        required: true
        type: "choice"

jobs:
  deploy-project-artifact:
    uses: "augustash/workflows/.github/workflows/magento2/deploy_artifact.yml@v1"
    with:
      deploy-environment: "${{ github.event.inputs.deploy-environment }}"
      deploy-target: "${{ github.event.inputs.deploy-target }}"
      languages: "en_US"
      php-version: "8.1"
      working-directory: "${{ github.workspace }}/src"
    secrets:
      composer-auth: "${{ secrets.COMPOSER_AUTH }}"
      deploy-ssh-host: "${{ secrets.DEPLOY_SSH_HOST }}"
      deploy-ssh-key: "${{ secrets.DEPLOY_SSH_KEY }}"
      github-token: "${{ secrets.GITHUB_TOKEN }}"
```

This is an advanced example of using a workflow for both an automatic and manual deployment.

```yaml
# File: .github/workflows/workflow.yml
name: "Project deployment"
run-name: "Running deployment by @${{ github.actor }} 🚀"

on:
  push:
    tags:
      - "v[0-9]+.[0-9]+.[0-9]+.[0-9]+"
  workflow_dispatch:
    inputs:
      deploy-environment:
        description: "GitHub environment to run against"
        required: true
        type: environment
      deploy-target:
        description: "The deployment target environment"
        options:
          - "staging"
          - "production"
        required: true
        type: "choice"
      languages:
        default: "en_US"
        required: true
        type: string

concurrency: "staging_environment"

jobs:
  resolve-input-parameters:
    runs-on: "ubuntu-latest"
    outputs:
      deploy-environment: "${{ steps.parameters.outputs.dep-environment }}"
      deploy-target: "${{ steps.parameters.outputs.dep-target }}"
      languages: "${{ steps.parameters.outputs.dep-languages }}"
    steps:
      - name: "Use shell parameter expansion to resolve input parameters"
        id: "parameters"
        shell: "bash"
        run: |
          USER_ENV=${{ github.event.inputs.deploy-environment }}
          USER_LANG=${{ github.event.inputs.languages }}
          USER_TARGET=${{ github.event.inputs.deploy-target }}
          echo "dep-environment=${USER_ENV:-Staging}" >> $GITHUB_OUTPUT
          echo "dep-languages=${USER_LANG:-en_US}" >> $GITHUB_OUTPUT
          echo "dep-target=${USER_TARGET:-staging}" >> $GITHUB_OUTPUT
  deploy-staging-environment:
    needs: "resolve-input-parameters"
    uses: "augustash/workflows/.github/workflows/magento2_deploy_artifact.yml@release/1.0"
    with:
      deploy-environment: "${{ needs.resolve-input-parameters.outputs.deploy-environment }}"
      deploy-target: "${{ needs.resolve-input-parameters.outputs.deploy-target }}"
      languages: "${{ needs.resolve-input-parameters.outputs.languages }}"
      php-version: "8.1"
      working-directory: "/src"
    secrets:
      composer-auth: "${{ secrets.COMPOSER_AUTH }}"
      deploy-ssh-host: "${{ secrets.DEPLOY_SSH_HOST }}"
      deploy-ssh-key: "${{ secrets.DEPLOY_SSH_KEY }}"
      github-token: "${{ secrets.GITHUB_TOKEN }}"

```

## 📝 License

The scripts and documentation in this project are released under the [MIT License](LICENSE).
