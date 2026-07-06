# Contributing

Thanks for helping improve the C-DAC PARAM Rudra (20 PF) user manual!

## How changes are accepted

**Direct pushes to `main` are disabled.** Every change — from anyone — lands
through a **reviewed pull request**. This keeps the published manual accurate and
reviewable.

The `main` branch is protected:

- No direct commits or force-pushes to `main`.
- Changes must come via a pull request.
- At least **one approving review** is required before merge.
- Open review conversations must be resolved before merge.

## Workflow

1. **Fork** the repository (button at the top-right on GitHub), or — if you are a
   collaborator — create a **branch**. External contributors do not have write
   access and must fork.
2. Make your edits under `docs/` (or `mkdocs.yml` for structure/theme).
3. (Optional) Preview locally:
   ```bash
   pip install -r requirements.txt
   mkdocs serve      # http://127.0.0.1:8000
   ```
4. **Open a pull request** against `main`. Fill in the PR template.
5. A maintainer reviews and merges. On merge, GitHub Actions rebuilds and
   redeploys the site automatically.

## Guidelines

- Verify commands, paths and values against the **live PARAM Rudra system** or
  the **official C-DAC manual** before documenting them.
- Keep internal links and page anchors working.
- **Never include secrets** (passwords, SSH private keys, access tokens) in
  content, issues, or pull requests. If a credential is ever exposed, rotate it
  immediately.

## Reporting issues

Spot something wrong but can't fix it yourself? Open an
[issue](https://github.com/samcom12/paramrudra-user-manual/issues) describing the
problem and the page it's on.
