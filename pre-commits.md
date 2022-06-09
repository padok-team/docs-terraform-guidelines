# Pre-commits for terraform

You should add pre-commits to your terraform repositories to have a higher minimum quality over your commits.

## Dependencies

In order to use pre-commit with all recommanded features, you will need to install some tools:
- [python](https://www.python.org/downloads/) and [pip](https://pip.pypa.io/en/stable/installation/)
- [terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- [tflint](https://github.com/terraform-linters/tflint#tflint)
- [terraform-docs](https://github.com/terraform-docs/terraform-docs#terraform-docs)
- [tfsec](https://github.com/aquasecurity/tfsec#installation)

    ```bash
    #tflint
    curl https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash


    #tfsec
    curl -kLs "https://github.com/aquasecurity/tfsec/releases/download/v1.23.2/tfsec-linux-amd64" > /usr/local/bin/tfsec
    chmod +x /usr/local/bin/tfsec

    #terraform-docs
    curl -Lo ./terraform-docs.tar.gz https://github.com/terraform-docs/terraform-docs/releases/download/v0.16.0/terraform-docs-v0.16.0-$(uname)-amd64.tar.gz
    tar -xzf terraform-docs.tar.gz
    chmod +x terraform-docs
    mv terraform-docs /usr/local/bin/terraform-docs

    ```

## Installation


1. Install the [`pre-commit`](https://pre-commit.com/) tool

    ```bash
    pip install pre-commit
    ```
2. Install the git hook scripts

    ```bash
    pre-commit install
    ```
3. create a `.pre-commit-config.yaml`
    ```bash
    #.pre-commit-config.yaml
    ---
    repos:
    - repo: https://github.com/pre-commit/pre-commit-hooks
        rev: v4.2.0
        hooks:
        - id: check-added-large-files
        - id: check-json
        - id: pretty-format-json
        - id: check-yaml
        - id: end-of-file-fixer
        - id: trailing-whitespace
    - repo: https://github.com/jumanjihouse/pre-commit-hook-yamlfmt
        rev: 0.1.1
        hooks:
        - id: yamlfmt
            args: [--mapping, '2', --sequence, '4', --offset, '2']
    - repo: https://github.com/antonbabenko/pre-commit-terraform
        rev: v1.71.0
        hooks:
        - id: terraform_fmt
        - id: terraform_validate
        - id: terraform_tflint
        - id: terraform_tfsec
        - id: terraform_docs
    - repo: https://github.com/thlorenz/doctoc
        rev: v2.2.0
        hooks:
        - id: doctoc
    ```

## Usage

Pre-commit will only check files pending for commit.

    ```bash
    # With one file
    git add myfile.tf
    pre-commit

    # Run on all changed files
    pre-commit run -a
    ```

### Update

Run `pre-commit autoupdate` in order to bump pre-commit repositories tag versions
