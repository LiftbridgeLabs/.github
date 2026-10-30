# .github
Organization profile and shared community files for Liftbridge Labs

## Security scans

`.github/workflows/security-scan.yml` is the one shared definition of our
standard scans. Every repo calls it from a small `.github/workflows/security.yml`
of its own:

- **secrets**: gitleaks over the whole git history (pinned binary, checksum checked)
- **npm-audit**: known vulnerabilities in all npm dependencies, build tools included (high and critical fail)
- **pip-audit**: known vulnerabilities in Python requirements files
- **new-deps**: on every push and pull request, finds the dependency names it
  adds (npm, PyPI, Maven, Swift packages and GitHub Actions, indirect ones in
  lockfiles included; version bumps are not new names) and looks each one up
  on its registry. The log lists its link, owner, first-published date and
  usage. It fails when a name isn't on the registry or was first published
  under 30 days ago, which is how invented or freshly squatted packages look.
  Every repo gets it with no setup. What it can't check is written above the
  job in the workflow.
- **alert**: posts to Discord when a scan fails (needs the `ALERTS_DISCORD_WEBHOOK` organization secret)

When new-deps fails, remove or replace the package. If Jason has looked at it
and said yes (a package under 30 days old he wants, or one from a source the
check can't look up), add a line to the calling repo's
`.github/accepted-dependencies.txt`: the kind (`npm`, `pypi`, `maven`, `swift`
or `actions`), the name as the log shows it, and the reason, for example
`npm some-package Jason OK'd Oct 30: needed for the new player`. A line with
no reason is ignored.

To add a repo, copy `security.yml` from any other repo and set `npm-dirs`
(folders holding a `package-lock.json`) and `pip-requirements` where they apply.
GitHub does not pass secrets to a called workflow on its own, so the caller maps
the organization secret to the name the workflow reads:
`DISCORD_WEBHOOK: ${{ secrets.ALERTS_DISCORD_WEBHOOK }}` (already in the copied file).
Sites and images that deploy automatically also call the scan first in their
deploy workflow, so a failing scan stops the deploy.

To check that alerts still reach Discord, open the repo's Security workflow
in the Actions tab, choose **Run workflow** and tick **Send a test alert**.
The message says it is a test. A repo's `security.yml` needs the `test-alert`
input for the box to appear (liftbridgelabs.app's has it; copy from there).
