# Deployment & GitHub Pages guidance

This repository uses a GitHub Actions workflow to run `bundle exec jekyll build` and deploy
the generated site artifact to GitHub Pages.

Recommendations

- Preferred: configure **GitHub Pages** to use the **GitHub Actions** deployment (the
  action-produced artifact is authoritative and avoids serving raw repository files).
- Do not commit the generated output (files from `_site/`, an `index.html` rendered by
  the build, or other generated assets) into the `main` branch. These files are build
  artifacts and can confuse Pages or override generated content.
- Keep `_site/` and other build output in `.gitignore` (this repo already ignores `_site/`).

How to fix if the public site serves raw front matter

1. Remove or revert the accidentally committed generated files (e.g., `index.html`).
2. Re-run the Actions workflow (or push a small change) to rebuild and redeploy the artifact.
3. Ensure Pages is configured to use the Actions workflow as the publishing source.

If you'd like, I can:
- Switch Pages to use the workflow artifact (requires repo admin/API permissions), or
- Revert the commit that added generated files and restore the source index front-matter.

If you're happy with the current quick fix, you can keep the generated `index.html` for now and
schedule the cleanup later — either way, the guidance above will reduce the chance of a
recurrence.
