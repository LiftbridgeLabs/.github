# .github
Organization profile and shared community files for Liftbridge Labs

## Security scans

`.github/workflows/security-scan.yml` is the one shared definition of our
standard scans. Every repo calls it from a small `.github/workflows/security.yml`
of its own:

- **secrets**: gitleaks over the whole git history (pinned binary, checksum checked)
- **npm-audit**: known vulnerabilities in shipped npm dependencies (high and critical fail)
- **pip-audit**: known vulnerabilities in Python requirements files
- **alert**: posts to Discord when a scan fails (needs the `ALERTS_DISCORD_WEBHOOK` organization secret)

To add a repo, copy `security.yml` from any other repo and set `npm-dirs`
(folders holding a `package-lock.json`) and `pip-requirements` where they apply.
GitHub does not pass secrets to a called workflow on its own, so the caller maps
the organization secret to the name the workflow reads:
`DISCORD_WEBHOOK: ${{ secrets.ALERTS_DISCORD_WEBHOOK }}` (already in the copied file).
Sites and images that deploy automatically also call the scan first in their
deploy workflow, so a failing scan stops the deploy.
