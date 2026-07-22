# RL QP Controller documentation

Documentation website for the RL QP Controller, built with MkDocs Material and deployed with GitHub Pages.

## Local preview

The documentation is served through Docker, so no local Python environment or manual dependency installation is required.

From the repository root, run:

```bash
cd docs
make
```

The first invocation builds the documentation image. Once MkDocs is running, open <http://localhost:8000>. Changes made under `docs/` are reloaded automatically.

Other available commands are:

```bash
make build      # Build the static website
make version    # Display the MkDocs version used by the image
make help       # Display the MkDocs command help
```

## Placeholder project links

Update `site_url`, `repo_url`, `edit_uri`, `extra.header_links`, and `extra.footer_links` in `docs/mkdocs.yml` before publishing.
