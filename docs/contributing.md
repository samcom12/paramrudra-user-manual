# Contributing

This manual is a **living document** — corrections and additions from users and
staff are welcome. To keep the published content accurate, **all changes go
through a reviewed pull request**; direct pushes to `main` are disabled.

!!! info "How the review gate works"
    Every change lands via a pull request that must (1) **pass the automated
    build check** and (2) receive **one approving review** from a maintainer,
    with all review conversations resolved. On merge, the site and PDF
    **rebuild and redeploy automatically** within a couple of minutes.

Pick the path that fits the size of your change.

## Path A — Quick edit in the browser

Best for small text fixes. No git install needed — GitHub handles the fork for
you.

1. Open the live page you want to change, e.g.
   `https://samcom12.github.io/paramrudra-user-manual/batch/`.
2. Click the **:material-pencil: (Edit this page)** icon at the top-right — it
   jumps to that file on GitHub.
3. Click the **pencil / "Edit this file"** button. GitHub shows *"You need to
   fork this repository to propose changes"* → click **Fork this repository**.
4. Edit the Markdown in the browser.
5. Scroll down, write a short commit message, and choose **"Create a new branch
   and start a pull request"**.
6. Click **Propose changes**, then **Create pull request**, fill in the template,
   and submit.

## Path B — Fork + local

Best for larger edits, new pages, or previewing before you submit.

### One-time setup

1. **Fork** the repository — on
   [github.com/samcom12/paramrudra-user-manual](https://github.com/samcom12/paramrudra-user-manual)
   click **Fork** (top-right).
2. **Clone** your fork:
   ```bash
   git clone https://github.com/<your-username>/paramrudra-user-manual.git
   cd paramrudra-user-manual
   ```
3. *(Optional)* add the original repo as `upstream` to stay in sync:
   ```bash
   git remote add upstream https://github.com/samcom12/paramrudra-user-manual.git
   ```

### For each change

4. Create a branch:
   ```bash
   git checkout -b fix/batch-typo
   ```
5. Edit files under `docs/` (content) or `mkdocs.yml` (structure/nav).
   **New page →** add the `.md` under `docs/` **and** add it to `nav:` in
   `mkdocs.yml`.
6. *(Recommended)* preview locally:
   ```bash
   pip install -r requirements.txt
   mkdocs serve        # open http://127.0.0.1:8000
   ```
7. Commit and push to **your fork**:
   ```bash
   git add -A
   git commit -m "Fix typo on batch page"
   git push origin fix/batch-typo
   ```
8. On GitHub, open a **pull request** from your branch → base repo
   `samcom12/paramrudra-user-manual`, base branch `main`. Fill in the PR
   template.

## What happens after you open a pull request

Both paths converge here:

1. The **PR Build Check** (`docs-build`) runs automatically — it must go
   :material-check-circle:{ style="color:#2e7d32" } **green** (the docs must
   build).
2. A maintainer is **auto-requested as reviewer** (via `CODEOWNERS`). They
   review, may request changes, then **Approve**.
3. Resolve any review conversations, then the PR can be **merged** — only once
   the build passed, one approval is in, and conversations are resolved.
4. On merge to `main`, GitHub Actions **rebuilds and redeploys** the site and PDF
   automatically. Your change is live in ~1–2 minutes.

## Style & content guidelines

- **Verify before you document.** Check commands, paths, versions and values
  against the **live PARAM Rudra system** or the **official C-DAC manual**.
- **Keep links working.** Internal links and page anchors must still resolve.
  New pages must be added to `nav:` in `mkdocs.yml`.
- **Match the house style.** Use admonitions (`!!! note`, `!!! warning`), fenced
  code blocks with a language, and tables — consistent with existing pages.
- **Prefer explicit versions/hashes** in examples (e.g. `spack load pkg /hash`)
  so instructions stay reproducible.

!!! danger "Never commit secrets"
    Do not include passwords, SSH private keys, OTP seeds or access tokens in
    content, commits, issues, or pull requests. If a credential is ever exposed,
    **rotate it immediately**.

## Reporting an issue

Spotted something wrong but can't fix it yourself? Open an
[issue](https://github.com/samcom12/paramrudra-user-manual/issues) describing the
problem and the page it's on — that's a valuable contribution too.

Thank you for helping keep the PARAM Rudra user guide accurate and useful!
