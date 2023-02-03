# Pre-commits for terraform

You should add pre-commits to your terraform repositories to have a higher minimum quality over your commits.

## Dependencies

In order to use pre-commit with all recommanded features, you will need to install some tools

- [python](https://www.python.org/downloads/) and [pip](https://pip.pypa.io/en/stable/installation/)
- [terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- [tflint](https://github.com/terraform-linters/tflint#installation)
- [terraform-docs](https://github.com/terraform-docs/terraform-docs#installation)
<!-- - [tfsec](https://github.com/aquasecurity/tfsec#installation) -->
<!-- - [checkov](https://github.com/bridgecrewio/checkov#installation) -->

## Installation

1. Install the [`pre-commit`](https://pre-commit.com/) tool

    ```bash
    pip install pre-commit
    ```

2. Install the git hook scripts

    ```bash
    pre-commit install
    ```

3. Copy the [`.pre-commit-config.yaml` file](../assets/.pre-commit-config.yaml) at the root of your repository

## Usage

Pre-commit will only check files pending for commit.

    ```bash
    # With one file
    git add myfile.tf
    git commit # will trigger pre-commit scripts

    # To trigger it manually
    pre-commit

    # Run on all changed files
    pre-commit run -a
    ```

### Update

Run `pre-commit autoupdate` in order to bump pre-commit repositories tag versions
