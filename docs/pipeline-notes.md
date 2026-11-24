# Pipeline documentation — source notes

This file is the running record of everything provided for the CI/CD documentation. It is the source the finished documents will be written from. Nothing here is invented, and nothing provided is dropped.

## Recording rules

- Every input gets a numbered entry, dated to the minute, and is stored **verbatim** — including typos, spacing, and formatting.
- My own remarks sit on a separate `Note:` line, always outside the quoted material.
- Links, file paths, commands, and numbers are kept exactly as given.
- If an input is unclear, I ask rather than paraphrase it into something clearer.

## Decisions on record

| Date | Decision | How it was given |
|---|---|---|
| 2026-11-24 | The application pipeline runs from `banking-system` in a GitHub repository the project owner controls | Selected in reply to "Where will the application pipeline run from?" |
| 2026-11-24 | Deployment target is EKS through ArgoCD | Selected in reply to "What should the last stage of the pipeline do?" |
| 2026-11-24 | Images are published to Amazon ECR | Selected in reply to "Which registry should the images go to?" |

## Entry log

### Entry 1 — 2026-11-24 02:38 WAT — Documentation outline

Source: chat message

```
Table of Contents
Prerequisites
Introduction
How to Set Up Your Source Control and Project Structure
How to Build Your CI Pipeline with GitHub Actions
How to Optimize Docker Builds for CI
Infrastructure as Code Using Terraform and Free Cloud Providers
How to Set Up Container Orchestration on Minimal Resources
How to Create a Deployment Pipeline
How to Build a Comprehensive Monitoring System
How to Implement Security Testing and Scanning
Performance Optimization and Scaling
Putting it All Together
Conclusion
```

Note: recorded as one outline, in the given order, including the "Table of Contents" line. No section has content yet.

### Entry 2 — 2026-11-24 02:50 WAT — Source control setup

Source: chat message

```
First of all, I cloned the repo: https://github.com/Kenechukwu-Okeke/Banking_System.git, then i, Set Up Branch Protection for the main branch, to Require pull requests before merging, Enable status checks to prevent broken code from landing in main in the future, Enforce linear history for cleaner version control.  (For Deepseek:  ensure to add reasons why thiis points are inportant in the documentation)
```

Note: spelling, double spacing, and the parenthetical are preserved exactly as given. The parenthetical is an instruction rather than a description of the work: the documentation must explain why each of the three protections matters, not merely list what was enabled.

### Entry 3 — 2026-11-24 03:10 WAT — Ruleset configured and push test

Source: chat message with three screenshots

```
And to be sure it works I added, i testing.txt file and tried to push to the main branch, but it stopped the push. So i now have to continue with a new branch "Devops"
```

Screenshots saved to `docs/assets/`:

| File | Shows |
|---|---|
| `01-ruleset-definition.webp` | Ruleset creation page: name, enforcement status, bypass list, target branches, branch rules |
| `02-ruleset-rules.webp` | Rule details: pull request requirements, merge methods, status checks, force pushes |
| `03-push-blocked.webp` | Terminal: rejected push to `main` with GH013 |

Observed in the screenshots:

- Ruleset name `Main_Branch_Ruleset`, enforcement `Active`, bypass list empty, target branch `main`.
- `Restrict deletions` checked; `Restrict creations` and `Restrict updates` unchecked.
- `Require a pull request before merging` checked, with: required approvals 1, dismiss stale approvals on new commits, require conversation resolution, require an additional approval for unattributed Copilot pull requests. Requiring review from specific teams, from Code Owners, and approval of the most recent reviewable push are unchecked.
- Allowed merge methods: `Merge, Squash, Rebase`.
- `Require status checks to pass` unchecked.
- `Block force pushes` checked.
- Push test: commit `added testing file` created on `main`, push rejected with `GH013: Repository rule violations found for refs/heads/main` and `- Changes must be made through a pull request.`

Note: the recorded screenshots show status checks still disabled, so the documentation must not claim that check is enforcing yet. All three merge methods are allowed, which permits merge commits and therefore does not by itself produce a linear history.

Note: the record of this step ends with the intent to continue the work on a branch named `Devops`.

### Entry 4 — 2026-11-24 03:13 WAT — Why status checks are deferred

Source: chat message

```
yeah , i will added Status checks after, i removed it now because i removed the inital pipeline and the merges might not go through if the new pipeline is not ready
```

Note: spelling preserved. This is the reason the documentation should give for the sequencing: a required check that no workflow reports leaves every pull request stuck at "Expected", so the requirement goes on only after the replacement workflow exists and has completed a run.

### Entry 5 — 2026-11-24 12:04 WAT — Pre-commit hooks with Husky and Gitleaks

Source: chat message, including commands and config files intended for the repo

Next up Advanced Technique: Set Up Pre-Commit Hooks with Husky and Gitleaks

Before code ever hits GitHub, i can catch issues locally with Git hooks. Using Husky to manage the hook and Gitleaks to scan for secrets, you can:

- Block API keys, passwords, and private keys from being committed
- Share the same hook with every teammate automatically via npm install
- Back it up with a CI check so the hook can't be bypassed

```bash
# Install Husky at the repo root (banking-system/)
npm install --save-dev husky
npx husky init

# Install Gitleaks locally (one-time, per developer)
# macOS:   brew install gitleaks
# Windows: winget install gitleaks
# Linux:   download from https://github.com/gitleaks/gitleaks/releases
```

Then replace the contents of .husky/pre-commit so it scans staged files and blocks the commit if anything is found:

```sh
if ! command -v gitleaks >/dev/null 2>&1; then
  echo "✖ gitleaks is not installed. See SETUP.md (Prerequisites)."
  exit 1
fi

gitleaks git --pre-commit --staged --redact -v || exit 1
```

Add a .gitleaks.toml at the root to allowlist known false positives:

```toml
[extend]
useDefault = true

[allowlist]
paths = [
  '''\.env\.example$''',

  '''package-lock\.json$''',
]
```

Finally, enforce the same check on the server so git commit --no-verify can't skip it. Create .github/workflows/gitleaks.yml:

```yaml
name: gitleaks
on: [pull_request, push]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

```
(For Deep seek: I have added Require status checks to pass before merging in the ruleset.)
```

Note: the parenthetical is a status change, not description of the hook work. It supersedes the deferral recorded in Entry 4: status checks are now enabled on the ruleset.

### Entry 6 — 2026-11-24 12:10 WAT — Publishing platform

Source: chat message

```
For Deepseek: The platform i am going to publiish the documentation is Hashnode. SO have that in mind
```

Note: spelling preserved. The documentation is to be published on Hashnode, which constrains the output:

- Markdown, GitHub-flavoured, with language-tagged code fences; no MDX components or raw HTML widgets.
- Images must resolve from a public URL. Relative repo paths such as `docs/assets/01-ruleset-definition.webp` will not render; the files are kept here as the source, but each needs to be uploaded through Hashnode's uploader (or served from a public host) when the post is assembled. Cover images must come from Hashnode's CDN.
- Tables and headings are supported, and headings drive Hashnode's generated table of contents.
- The outline has thirteen sections; whether it ships as one long post or a Hashnode series is still open.
