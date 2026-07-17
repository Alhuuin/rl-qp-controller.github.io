---
title: "MkDocs Material Template"
---
# MkDocs Material Template
This is an MkDocs Material Template repository. It enables a useful set of extensions and plugins by default, and contains a Makefile that uses Docker to serve the documentation locally, and a Github workflow to publish the documentation on Github Pages by default.

To use, invoke:

```bash
make
```

This will publish the documentation at [localhost:8000](http://localhost:8000/) by default, and watch for any changes to the documentation source files. Other targets in the `Makefile` include:

- `serve` (default) — Serves the documentation and watches for changes.
- `build` — Builds the documentation.
- `gh-deploy` — Deploys the documentation to Github Pages.
- `version` — Prints the version information.
- `help` — Prints help information.

[MkDocs Material Documentation](https://squidfunk.github.io/mkdocs-material/){ .md-button }


## Customization
Change the fields in `mkdocs.yml` to suit your needs. In particular, change the `site_name`, `site_description` and `site_author`, and the URLs for `repo_url` (where the Github button links to) and `edit_uri` (where the pages can be edited).


## Adding Plugins
A few plugins have been enabled by default. To add additional plugins, add them to the `plugins` section of `mkdocs.yml` ([documentation](https://www.mkdocs.org/user-guide/configuration/#plugins)) and the required packages to `requirements.txt`. The plugins are automatically installed into the Docker image.


## Updating MkDocs Material
To change the version of MkDocs Material, change the version number in `requirements.txt`. The version is automatically updated in the Docker image as well.


## Serving on a Different Port
By default the documentation is served on port `8000`. To change the default temporarily, invoke `make` like this:

```bash
make PORT=8080
```

To permanently chang the port, change the value of `PORT` in the `Makefile`.

```Makefile
PORT ?= 8080
```


## Makefile on Top-Level
If you want to have the `Makefile` on the top level instead of in the `docs/` directory, move it to the top level directory and change the `ROOT` variable in the `Makefile` to:

```Makefile
ROOT ?= $(DIR)/.
```

!!! note ""
    Note the slash-dot `/.` at the end. The `ROOT` directory points to the directory with `mkdocs.yml` relative to the `Makefile`.


## Different branch
The template assumes your default branch is named `main` or `master`. If it's named something else, change the `on push branches` in `.github/workflows/docs.yml`:

```yaml
on:
  push:
    branches:
      - my-docs-branch
```

# Deploying
This documentation can be deployed on Github (`.github/workflows/documentation.yml`)
or Gitlab (`.gitlab-ci.yml`).


# Updating to Latest Template
This page describes how to update your documentation to include the latest
changes to the template repository.


## Template Remote
To update to the latest version of the template, first ensure the template
repository is added as a remote for the documentation repository:

```sh
git remote -v

origin      git@github.com:MyUsername/my-docs-repo.git (fetch)
origin      git@github.com:MyUsername/my-docs-repo.git (push)
template    git@github.com:Virtlink/mkdocs-material-template.git (fetch)
template    git@github.com:Virtlink/mkdocs-material-template.git (push)
```

If not, add the template remote repository:

```sh
git remote add template git@github.com:Virtlink/mkdocs-material-template.git
```


## Fetch Latest Changes
Fetch the changes in the template remote:

```sh
git fetch template
```


## Merge Latest Changes
Merge the latest changes from the template's `main` branch into your repository:

```sh
git merge template/main
```

??? tip "Unrelated Histories"
    If you get the error `fatal: refusing to merge unrelated histories`, then
    your documentation repository was never based on the template. However, you
    can force Git to merge the template anyway by using this command:

    ```sh
    git merge template/main --allow-unrelated-histories
    ```


## Fix Any Conflicts, and Commit
Now use the Git command line to fix any merge conflicts, and commit the result.


## Fix the Breaking Changes
New versions of MkDocs and MkDocs Material can have breaking changes.
[Find them here](https://squidfunk.github.io/mkdocs-material/upgrade/#changes-to-mkdocsyml).