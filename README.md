> [!WARNING]
> _MkDocs for Material is [deprecated](https://github.com/squidfunk/mkdocs-material/issues/8523)._
> For new projects, use [Zensical](https://zensical.org/) documentation instead.
> Here is the [template for Zensical documentation projects](https://github.com/Virtlink/zensical-docs-template).

# MkDocs Material Template
[![Build][github-build-badge]][github-build]
[![License][license-badge]][license]
[![Documentation][documentation-badge]][documentation]

A MkDocs Material Template repository. It enables a useful set of extensions and plugins by default, and contains a Makefile that uses Docker to serve the documentation locally, and a GitHub workflow to publish the documentation on GitHub Pages by default.

[![Documentation][documentation-button]][documentation]

- [Getting Started][documentation]
- [Material for MkDocs documentation](https://squidfunk.github.io/mkdocs-material/)

This template is intended both for those that include the documentation in their main repository, and those that use a separate repository for the documentation.


## Quick Start
To build the pages and see edits live using [Docker](https://www.docker.com/):

```shell
cd docs/
make
```

Or using [Python 3](https://www.python.org/), creating and activating a _virtual environment_:

```shell
cd docs/

python3 -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt
mkdocs serve

deactivate
```

Or using [UV](https://docs.astral.sh/uv/), creating and activating a _virtual environment_:

```shell
cd docs/

uv venv

uv pip install -r requirements.txt
uv run mkdocs serve
```

Navigate to [localhost:8000](http://localhost:8000/) to see the documentation.
The local documentation is automatically reloaded when changes occur.
Changes pushed to the `main` branch are automatically deployed to GitHub Pages.


## Updating Dependencies
Install the [`pip-check-updates`](https://pypi.org/project/pip-check-updates/) tool in a _virtual environment_ (e.g., using `pip install pip-check-updates` or `uv pip install pip-check-updates`). Usage:

```shell
cd docs/
pcu requirements.txt
```

And update the dependencies to their latest versions using:

```shell
cd docs/
pcu -u requirements.txt
```

> [!NOTE]
> Install the updated dependencies using:
>
> ```shell
> pip install -r requirements.txt
> ```
>
> Or, when using UV:
>
> ```shell
> uv pip install -r requirements.txt
> ```


## License
[![License: CC0-1.0](https://licensebuttons.net/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, Daniel A. A. Pelsmaeker has waived all copyright and related or neighboring rights to the MkDocs Material Template repository. Feel free to use this as you see fit, no attribution required.

[github-build-badge]: https://img.shields.io/github/actions/workflow/status/Virtlink/mkdocs-material-template/documentation.yml
[github-build]: https://github.com/Virtlink/mkdocs-material-template/actions
[license-badge]: https://img.shields.io/github/license/Virtlink/mkdocs-material-template
[license]: https://github.com/Virtlink/mkdocs-material-template/blob/master/LICENSE
[documentation-badge]: https://img.shields.io/badge/docs-latest-brightgreen
[documentation]: https://pelsmaeker.net/mkdocs-material-template/
[documentation-button]: https://img.shields.io/badge/Documentation-blue?style=for-the-badge&logo=googledocs&logoColor=white
