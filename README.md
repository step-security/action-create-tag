[![StepSecurity Maintained Action](https://raw.githubusercontent.com/step-security/maintained-actions-assets/main/assets/maintained-action-banner.png)](https://docs.stepsecurity.io/actions/stepsecurity-maintained-actions)

# GitHub Action: Create/update tag

[![Docker Image CI](https://github.com/step-security/action-create-tag/workflows/Docker%20Image%20CI/badge.svg)](https://github.com/step-security/action-create-tag/actions)
[![Code quality CI](https://github.com/step-security/action-create-tag/workflows/Code%20quality%20CI/badge.svg)](https://github.com/step-security/action-create-tag/actions?query=workflow%3A%22Code+quality+CI%22)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/step-security/action-create-tag?logo=github&sort=semver)](https://github.com/step-security/action-create-tag/releases)

Simple (docker-based) GitHub action that can be used to create/update a tag and push it to the remote.

> [!NOTE]\
> Since this is a docker-based action and [GitHub currently only supports Linux-based containers](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action) running this action on Windows and Mac now needs to be supported

## Inputs

### `tag`

**Required**. Tag you want to create.

### `message`

**Optional**. Tag message. Default: `Release $TAG`.

### `force_push_tag`

**Optional**. Push tag even if it already exists on the remote. Default: `false`. Please use with care!

### `tag_exists_error`

**Optional**. Whether to throw an error when the tag already exists. Default: `true`. Ignored when `force_push_tag` is `true`.

### `no_verify_tag`

**Optional**. Skips verifying when pushing the tag. Default: `false`. Please use with care!

### `commit_sha`

**Optional**. Specify the commit SHA hash for tagging. By default, it utilizes the `GITHUB_SHA` environment variable, which typically represents the latest commit SHA hash. However, its value ultimately depends on the trigger event of the workflow. For additional details, consult the [GitHub Actions documentation](https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#github-context).

### `github_token`

**Optional**. It's no need to specify it if you use checkout@v2. Required for
checkout@v1 action.

### `gpg_private_key`

**Optional**. Specifies the GPG private key to sign the tag with. Default: `""`.

### `gpg_passphrase`

**Optional**. Specifies the GPG passphrase to sign the tag with. Default: `""`.

## Outputs

### `tag_exists`

A boolean specifying whether the tag already exists.

## Environment variables

### `TAG_EXISTS`

A boolean specifying whether the tag already exists.

## Example Usage

### General example

```yml
name: Create/update tag
on:
  push:
    branch: "main"
jobs:
  create-tag:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - uses: step-security/action-create-tag@v1
        id: "tag_create"
        with:
          tag: "latest"
          tag_exists_error: false
          message: "Latest release"

      # Print result using the env variable.
      - run: |
        echo "Tag already present: ${{ env.TAG_EXISTS }}"

      # Print result using the action output.
      - run: |
        echo "Tag already present: ${{ steps.tag_create.outputs.tag_exists }}"
```

### Signing Tags with GPG

To sign tags with GPG, follow these steps:

#### 1. Generate a GPG Key

First, [generate a GPG key](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key). Once generated, export the GPG private key in ASCII armored format to your clipboard using one of the following commands based on your operating system:

- **macOS:**

  ```shell
  gpg --armor --export-secret-key joe@foo.bar | pbcopy
  ```

- **Ubuntu (GNU base64):**

  ```shell
  gpg --armor --export-secret-key joe@foo.bar -w0 | xclip -selection clipboard
  ```

- **Arch:**

  ```shell
  gpg --armor --export-secret-key joe@foo.bar | xclip -selection clipboard -i
  ```

- **FreeBSD (BSD base64):**

  ```shell
  gpg --armor --export-s[.github/workflows/update_semver.yml](.github/workflows/update_semver.yml)e your GPG passphrase.
  ```

#### 3. Update Workflow YAML

Modify your workflow YAML file to include the GPG private key and passphrase in the `gpg_private_key` and `gpg_passphrase` inputs:

```yaml
name: Create/update tag
on:
  push:
    branch: "main"
jobs:
  create-tag:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - uses: step-security/action-create-tag@v1
        id: "tag_create"
        with:
          tag: "latest"
          tag_exists_error: false
          message: "Latest release"
          gpg_private_key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg_passphrase: ${{ secrets.PASSPHRASE }}

      # Print result using the env variable.
      - run: |
        echo "Tag already present: ${{ env.TAG_EXISTS }}"

      # Print result using the action output.
      - run: |
        echo "Tag already present: ${{ steps.tag_create.outputs.tag_exists }}"
```

This workflow will now sign tags using the specified GPG key during tag creation.
