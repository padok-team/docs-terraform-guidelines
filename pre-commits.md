# Pre-commits for terraform

You should add pre-commits to your terraform repositories to have a higher minimum quality over your commits.

## Installation

1. Install the [`pre-commit`](https://pre-commit.com/) tool

    ```bash
    pip install pre-commit
    ```

2. Install the dependencies

    ```bash
    # tflint
    curl https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash
    ```

    <!-- TODO: Add tfsec etc. -->

3. Install the git hook scripts

    ```bash
    pre-commit install
    ```

<!-- TODO: Add installation of commit-msg stage -->

## Usage

Just commit and pre-commit hooks will be triggered.

If you want to manually run the scripts, run `pre-commit run -a`

### Update

Run `pre-commit autoupdate` in order to bump pre-commit repositories tag versions
