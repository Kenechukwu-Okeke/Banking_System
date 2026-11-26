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

### Entry 7 — 2026-11-24 15:09 WAT — First commit through the hook

Source: chat message with one screenshot

```
This is me using Gitleak to commit the recent changes and it made
```

Screenshot saved as `docs/assets/04-precommit-hook.webp`.

Observed in the screenshot:

- Command run from the repo root on branch `Devops`: `git commit -m "added .gitleaks.toml and  gitleaks.yml"`.
- Gitleaks printed its banner, then `1 commits scanned.`, `scan completed in 99.4ms`, `no leaks found`.
- The commit then completed: `[Devops e636de0] added .gitleaks.toml and  gitleaks.yml`, with 6 files changed, 2777 insertions(+), 1 deletion(-).
- Files created in that commit: `.github/workflows/gitleaks.yml`, `.gitleaks.toml`, `.husky/pre-commit`, `package-lock.json`.

Note: this shows the hook running before the commit object exists and a clean scan letting it through. The documentation should pair it with a blocked commit containing a fake secret, so the reader sees both outcomes.

### Entry 8 — 2026-11-24 15:13 WAT — Blocked commit with a demo secret

Source: chat message with one screenshot

```
This is me testing Gitleaks by trying to commit a file with a demo aws access token and git leaks didn't allow the commit to go through.
```

Screenshot saved as `docs/assets/05-precommit-blocked.webp`.

Observed in the screenshot:

- Commands: `echo 'AWS_SECRET_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE' > gitleaks-test.txt`, then `git add gitleaks-test.txt`, then `git commit -m "test gitleaks detection"`.
- Gitleaks reported the finding with `Secret: REDACTED`, `RuleID: aws-access-token`, `Entropy: 3.684184`, `File: gitleaks-test.txt`, `Line: 1`, `Fingerprint: gitleaks-test.txt:aws-access-token:1`.
- Summary lines: `1 commits scanned.`, `scan completed in 83ms`, `WRN leaks found: 1`.
- The commit was abandoned: `husky - pre-commit script failed (code 1)`, and the shell returned to the prompt with no commit created.

Note: together with Entry 7 this gives both outcomes — a clean scan committing in 99.4ms, and a leak rejected in 83ms. The `--redact` flag is what turns the secret into `REDACTED` in the finding, which matters because terminal and CI logs are retained.

Note: the test file `gitleaks-test.txt` appears to remain staged after the rejection; the documentation should tell the reader to remove it (`git rm --cached gitleaks-test.txt` or `git reset gitleaks-test.txt && rm gitleaks-test.txt`) so the demo does not pollute the next commit.

### Entry 9 — 2026-11-24 15:16 WAT — Why Gitleaks also runs in CI

Source: chat message

```
I added the gitleaks as a pipeline also because of these limitations:

there are limitations:
- A developer can bypass the hook using git commit --no-verify.
- A developer might forget to install Gitleaks.
- Someone could push code without using your configured local hook.
- Other contributors might not follow the same setup instructions.

It acts as an independent security check on GitHub regardless whether the developer installs gitleaks or forces a push and so on to by pass these limitations
```

Note: spelling preserved. This is the justification for the `gitleaks.yml` workflow and belongs in the same section as the hook: the hook is advisory because it lives on the developer's machine, the workflow is enforcement because it runs on GitHub and its result is a required status check.

### Entry 10 — 2026-11-24 15:46 WAT — CI pipeline

Source: chat message

```
I have set up this CI PIpeline:

name: CI Pipeline
on:
  push:
    branches:
      - main
      - devops
  pull_request:
    branches:
      - main
      - devops

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '24'
      - name: Cache node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies and run tests
        run: |
          npm ci
          npm test
          npm run lint




to puah to the devops and main branch
```

Note: spelling preserved, including "PIpeline" and "puah". The workflow triggers on pushes and pull requests targeting `main` and `devops`; the single job is named `build`, so the resulting status check is `build`.

Note: the steps as written do not match the repository. The root `package.json` has no `test` and no `lint` script, so `npm test` and `npm run lint` fail with "Missing script". `npm ci` at the root installs only the root dependencies; the backend and frontend have their own manifests, and neither is built or exercised by this job. The documentation must either add those scripts or replace the step with checks that exist, otherwise the workflow cannot go green and a required `build` check would block every pull request.

### Entry 11 — 2026-11-24 15:47 WAT — Dependency caching

Source: chat message

```
i added Cache your dependencies to increase speed and reduce installation time
```

Note: this is the stated reason for the `actions/cache` step in the workflow above. The section should explain what the cache actually holds — the `~/.npm` tarball cache rather than `node_modules` — because that is what makes it safe: npm verifies each cached package against the lockfile's integrity hash, so a stale cache entry is discarded rather than installed.

### Entry 12 — 2026-11-24 16:16 WAT — Dockerfiles rewritten

Source: instruction from the project owner (`For Deepseek: Rewrite the this frontend and backend docker files`). Implemented and verified locally.

The original files had four properties that do not belong in a production pipeline: the backend ran `npm audit fix --force` during the build, both images installed from `package.json` rather than the lockfile, both ran as root, and neither declared a healthcheck. The rewrite addresses each.

| Change | Reason |
|---|---|
| Node 18 → Node 24, both images | Node 18 reached end of life on 30 April 2025 and receives no security patches; 24 matches the CI workflow |
| `npm ci` from the lockfile | Installs are reproducible; the lockfile is authoritative instead of a range resolution performed at build time |
| `--omit=dev` in the backend deps stage | Dev dependencies stay out of the runtime image |
| `npm audit fix --force` removed | It mutates dependencies during the build, so the image stops matching the lockfile and can silently cross major versions |
| `.dockerignore` in both contexts | Smaller build context and faster builds; `.env` can no longer be copied into the image |
| Backend runs as `node` (uid 1000) | The process no longer runs as root, limiting what a compromised app can touch |
| Frontend runs as uid 101 on port 8080 | Uses `nginxinc/nginx-unprivileged`, so no root and no privileged bind to port 80 |
| `HEALTHCHECK` on both | Compose and orchestrators get a liveness signal from the same `/health` endpoint Kubernetes probes will use |
| Frontend API URL passed as a build ARG after `npm ci` | React bakes the value at build time, and declaring it after the dependency layer means a changed URL does not invalidate the install cache |

`backend/Dockerfile`:

```dockerfile
# syntax=docker/dockerfile:1

# ---------------------------------------------------------------------------
# Dependencies: installed once, from the lockfile, with dev packages excluded.
# ---------------------------------------------------------------------------
FROM node:24-alpine AS deps
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --omit=dev && npm cache clean --force

# ---------------------------------------------------------------------------
# Runtime: only the production dependencies and the server code.
# ---------------------------------------------------------------------------
FROM node:24-alpine
WORKDIR /app

ENV NODE_ENV=production \
    PORT=8080

COPY --from=deps --chown=node:node /app/node_modules ./node_modules
COPY --chown=node:node . .

# The image ships an unprivileged `node` user (uid 1000); nothing here needs root.
USER node

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- "http://127.0.0.1:${PORT}/health" >/dev/null || exit 1

CMD ["node", "server.js"]
```

`backend/.dockerignore`:

```
node_modules
npm-debug.log*
.env
.env.*
*.log
.git
.gitignore
Dockerfile
.dockerignore
README.md
```

`frontend/Dockerfile`:

```dockerfile
# syntax=docker/dockerfile:1

# ---------------------------------------------------------------------------
# Build: React compiles to static files. The API URL is baked in at build time,
# so it is passed as a build argument rather than an environment variable.
# ---------------------------------------------------------------------------
FROM node:24-alpine AS build
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

ARG REACT_APP_API_URL=""
ENV REACT_APP_API_URL=${REACT_APP_API_URL}

RUN npm run build

# ---------------------------------------------------------------------------
# Runtime: nginx without root, serving the static bundle and proxying /api.
# ---------------------------------------------------------------------------
FROM nginxinc/nginx-unprivileged:1.27-alpine

COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://127.0.0.1:8080/ >/dev/null || exit 1
```

`frontend/.dockerignore`:

```
node_modules
build
npm-debug.log*
.env
.env.*
*.log
.git
.gitignore
Dockerfile
.dockerignore
README.md
```

Supporting changes required by the new images:

- `frontend/nginx/default.conf` now listens on `8080` instead of `80`, because the unprivileged nginx image cannot bind below 1024.
- `docker-compose.yml` maps `3000:8080` instead of `3000:80` to match.
- `backend/package-lock.json` was generated and committed, because `npm ci` fails without a lockfile and the backend previously had none.

Verification performed:

- Both images build: `banking-backend:rewrite` and `banking-frontend:rewrite` (313 MB and 78.4 MB).
- Backend container starts against the Postgres instance, logs `Server running on port 8080`, answers `{"status":"ok"}` on `/health`, runs as uid 1000, and reports `healthy` to Docker.
- Frontend container serves the SPA on `/` and `/login` (`200`, correct title), proxies `/api/accounts` through to the backend (`401` without a session, which is the expected answer), runs as uid 101, and reports `healthy`.

Note: a container started with a different `DATA_ENCRYPTION_KEY` than the one used to write the existing account rows crashes on startup while decrypting them (`decryptAccountNumber` in `database/migrations.js`). The key therefore belongs in the secret store and must be treated as part of the data, not as an independent configuration value.

### Entry 13 — 2026-11-24 16:51 WAT — ECR pipeline and EKS manifests

Source: instruction from the project owner, with the technology list and the requested folder location. Designed and written in this session.

Scope requested: a CI pipeline that builds Docker images and pushes them to Amazon ECR, Kubernetes manifests that deploy the application to EKS, and the setup write-up. AWS access is through GitHub OIDC. The manifests live in `kubernetes/`, alongside `banking-system/` (Entry 14 records the move that produced this layout).

Files created:

| File | Role |
|---|---|
| `banking-system/.github/workflows/ci-ecr.yml` | Builds both images and pushes them to ECR with a `sha-<short>` tag |
| `kubernetes/00-namespace.yaml` | `banking` namespace |
| `kubernetes/01-storageclass.yaml` | gp3 EBS StorageClass (`ebs-gp3`) |
| `kubernetes/02-secret.yaml` | One secret for database credentials and application keys |
| `kubernetes/03-postgres-service.yaml` | Headless Service for stable database DNS |
| `kubernetes/04-postgres-statefulset.yaml` | Postgres 16 with a 10 Gi persistent volume |
| `kubernetes/05-backend-configmap.yaml` | Non-secret backend configuration |
| `kubernetes/06-backend-deployment.yaml` | Backend Deployment, two replicas |
| `kubernetes/07-backend-service.yaml` | Backend ClusterIP Service |
| `kubernetes/08-frontend-deployment.yaml` | Frontend Deployment, two replicas |
| `kubernetes/09-frontend-service.yaml` | Frontend ClusterIP Service |
| `kubernetes/10-ingress.yaml` | Hostname, TLS, `/api` and `/api-docs` routing |
| `kubernetes/11-argocd-application.yaml` | Argo CD Application tracking the folder |
| `kubernetes/README.md` | Setup guide, placeholder table, verification and troubleshooting |

Design decisions worth recording:

- **OIDC instead of access keys.** The workflow assumes `secrets.AWS_OIDC_ROLE_ARN`; the role trusts only `repo:<owner>/<repo>:ref:refs/heads/main` and `refs/heads/devops`. The README carries the trust policy and the minimal ECR policy.
- **Immutable SHA tags, not `latest`.** The ECR repositories are created with `image-tag-mutability IMMUTABLE` and `scanOnPush=true`. The tag in a manifest therefore identifies exactly one build, and every push is scanned by ECR at no extra cost.
- **GitOps through Argo CD.** The Application syncs `kubernetes/` with `prune` and `selfHeal`. A release is a commit that changes the image tag, which keeps `main` protected and the change auditable.
- **One secret, two consumers.** Postgres and the backend read the same `banking-secrets` keys, so the database credentials cannot drift between them.
- **The existing infrastructure is reused, not duplicated.** Ingress class `nginx`, ClusterIssuer `letsencrypt-prod`, the EBS CSI driver and Argo CD all come from the Terraform stack; the manifests only reference them.
- **Both pods run hardened.** Non-root, `allowPrivilegeEscalation: false`, all capabilities dropped, read-only root filesystem with `emptyDir` for `/tmp` (and `/var/cache/nginx`). Probes use the `/health` endpoint the backend already exposes.

Verification status, stated plainly: every YAML file parses, and both application images were built and run earlier (Entry 12). The manifests have **not** been applied to a cluster, because `kubectl` in this environment is a snap that cannot start inside the sandbox. The first real apply happens from the project owner's terminal.

Placeholders left for the owner to fill: `<AWS_ACCOUNT_ID>`, `<AWS_REGION>`, `<IMAGE_TAG>`, `<APP_HOSTNAME>`, `<GITOPS_REPO_URL>`, and the four secret values. The README lists each one with the command that produces it.

Note: the CI Pipeline workflow in Entry 10 still runs `npm test` and `npm run lint`, which do not exist in the root manifest. If its `build` check is required on `main`, every pull request blocks until that is fixed or the workflow is removed. The new workflow's job is named `build-and-push` so the two checks stay distinguishable.


### Entry 14 —  — Project relocated

Source: instruction from the project owner.

All work created for this documentation was moved out of the Terraform repository into a standalone project directory:

```
~/Projects/Banking-System-Architecture/
├── banking-system/     application repository (git history intact)
└── kubernetes/         manifests and setup guide
```

The Terraform stack stays where it was, at `Digital_Witch_Final_Project/Final-Project-architecture-EKS-terraform`, and is now a sibling project rather than a parent folder. Paths in `kubernetes/README.md` and in the earlier entries were corrected accordingly.

Note: the session's file-write policy still points at the old Terraform workspace, so edits in the new directory require an explicit approval per operation.


### Entry 15 —  — Kubernetes documentation page

Source: instruction from the project owner. The page was written in this session.

Requested: a detailed page on the `kubernetes/` folder covering structure, reasoning, decisions and comparisons, aimed at junior and mid-level DevOps engineers, published as a Notion page rather than as a Hashnode article. The project-directory move was to be left out of the text.

Deliverable: `documentation/01-kubernetes-deployment.md`, about 5,300 words in twelve sections.

Contents: the three workloads and what surrounds them; the request path from DNS to the database; the change path from commit to rollout; a file-by-file table explaining why each manifest exists; a short primer on labels, probes, requests and securityContext for readers new to Kubernetes; nine decision sections, each with the alternative and its trade-off (gp3 against gp2 and EFS, one Secret against four approaches to secret management, StatefulSet against RDS, rolling-update settings, the securityContext fields, ClusterIP and named ports, ingress-nginx against the AWS Load Balancer Controller, pull-based Argo CD against CI push); the placeholder table; a numbered deployment procedure in dependency order; day-two operations including release, rollback, scaling and secret changes; a troubleshooting table with three expanded entries; the gaps that would matter in production; and a glossary.

Note: this supersedes the platform decision recorded in Entry 6. The output format is Notion pages, so the image-CDN and single-post constraints no longer apply; the page is plain Markdown that imports into Notion as one page, with the section headings becoming its outline.

Note: the page states plainly that the manifests have not been applied to the cluster yet. The images have been built and run, and the YAML parses, but nothing in the document claims a successful cluster deployment.


### Entry 16 —  — Terraform configuration

Source: instruction from the project owner, after approving the architecture. The configuration was written and validated in this session.

Layout: `terraform/` holds three root modules rather than one.

| Module | Contents | State |
|---|---|---|
| `bootstrap/` | S3 state bucket: versioning, SSE-S3, public access blocked, policy denying non-TLS access | local, applied once |
| `infra/` | VPC with two public and two private subnets, one internet gateway, one NAT gateway, EKS cluster, managed node group, five add-ons, cluster/node/EBS-CSI/GitHub-CI IAM roles, two ECR repositories | `banking/infra/terraform.tfstate` |
| `platform/` | ingress-nginx, cert-manager, Let's Encrypt ClusterIssuer (local Helm chart), Argo CD | `banking/platform/terraform.tfstate` |

Decisions recorded from the answers given: two `t3.large` nodes, one NAT gateway, Terraform owns the ECR repositories, and the AWS account already has the GitHub OIDC provider so it is read as a data source rather than created.

Why three modules: the Helm and Kubernetes providers need the cluster endpoint and CA at plan time, which do not exist during the first apply of a single root module. `infra` creates the cluster, `platform` reads its state through `terraform_remote_state`. The ClusterIssuer ships as a small local chart because a `kubernetes_manifest` for a CRD fails at plan time on a fresh cluster.

State locking uses S3 native lock files (`use_lockfile = true`) rather than DynamoDB, which requires Terraform 1.10 or newer. The installed version is 1.17.2.

Consequential edits elsewhere:

- `banking-system/.github/workflows/ci-ecr.yml` no longer creates the repositories; Terraform owns them, so the workflow only logs in and pushes.
- `documentation/02-architecture.md` section 4 was rewritten for the three-module layout, and the state-key line was corrected.

Verification: `terraform fmt -check -recursive` is clean, and `terraform init -backend=false` followed by `terraform validate` succeeded in all three modules (providers downloaded, configuration valid). Nothing has been applied: no AWS credentials were used in this session, so the bucket, cluster and repositories do not exist yet. `infra/backend.tf` and `platform/backend.tf` still contain the placeholder bucket name until the owner runs the bootstrap step.


### Entry 17 —  — Terraform adjustments and upgrades

Source: instruction from the project owner, listing the changes to apply.

Applied changes:

| Setting | Before | Now |
|---|---|---|
| Region | `us-east-1` | `us-west-1` |
| Nodes | 2 × `t3.large` | 2 × `t3.medium` |
| NAT gateways | 1 | 1 (unchanged) |
| State bucket | placeholder | `banking-system-terraform-structure`, also used for the kubeconfig |
| GitHub OIDC | provider referenced | unchanged, referenced with `create_github_oidc_provider = false` |
| ECR | Terraform-owned | unchanged |
| DNS | Route 53 assumed | Namecheap, a manual CNAME |
| Node access | none | SSM Session Manager via `AmazonSSMManagedInstanceCore` on the node role |
| Terraform | `>= 1.10` | `>= 1.17` |
| AWS provider | `~> 5.60` | `~> 6.74` |
| Helm provider | `~> 2.14` | `~> 3.3` |
| Kubernetes provider | `~> 2.30` | `~> 3.5` |

New: `infra/kubeconfig.tf` renders a kubeconfig from the cluster endpoint and CA, writes it to `infra/kubeconfig.yaml` with mode 0600, and uploads it to `s3://banking-system-terraform-structure/kubeconfig/banking-eks.yaml`. The user entry uses `aws eks get-token`, so no static token is embedded. Outputs `kubeconfig_path` and `kubeconfig_s3_uri`.

The Helm provider upgrade was not a version bump alone: v3 rejects the `kubernetes { }` block and requires `kubernetes = { }` with a nested `exec = { }`. `terraform validate` caught it, and the provider block was rewritten.

Also added: `kubernetes/argocd/argocd-ingress.yaml`. Argo CD's Application syncs the top level of `kubernetes/` and does not recurse into subdirectories, so an Ingress for the `argocd` namespace can live there without Argo CD trying to manage it.

Documentation updated for the new values: the architecture page (region, sizing paragraph, DNS provider, kubeconfig row, and section 11 rewritten from open questions to the settled values), the Kubernetes page (region, cluster name, generated kubeconfig), `kubernetes/README.md` (Terraform path is now `../terraform`, output commands use `-chdir`), and `terraform/README.md` rewritten.

Verification: `terraform fmt -check -recursive` clean; `terraform init -backend=false` and `terraform validate` succeed in all three modules with the new provider versions; every manifest plus the new Argo CD Ingress parses. Still not applied: no AWS credentials were used, so the bucket, cluster and repositories do not exist yet.


### Entry 18 —  — Bootstrap module removed

Source: instruction from the project owner. The bucket already exists, so the module that created it was unnecessary complexity.

Removed: `terraform/bootstrap/` in full, including its local state file and provider lock.

The two remaining backends use the requested format, each with its own key:

```hcl
terraform {
  backend "s3" {
    bucket       = "banking-system-terraform-structure"
    key          = "banking/infra/terraform.tfstate"   # platform uses banking/platform/…
    region       = "us-west-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

Also removed: an unused `data "aws_caller_identity" "current"` in `infra/ecr.tf`, and the comments that referred to the removed module.

Updated: `terraform/README.md` rewritten for the two-module layout, with a short "before you start" section that states the bucket already exists and gives the one command to enable versioning. The architecture page now shows two modules, replaces the bootstrap row in the state table, and its build order starts at `terraform init` in `infra/`.

Consequence: bucket settings such as versioning, default encryption and public access blocking are no longer applied by Terraform. They are the owner's responsibility now. Versioning is optional for Terraform but is what makes a previous state object recoverable.

Verification: `terraform fmt -check -recursive` clean, and `terraform validate` succeeds in `infra/` and `platform/` after the deletion.


### Entry 19 —  — Terraform documentation section

Source: instruction from the project owner, asking for a comprehensive Notion-style section on the Terraform codebase.

Deliverable: a new top-level section at `documentation/terraform/`, eleven pages, about 18,600 words, every internal link verified. The existing pages (`01-kubernetes-deployment.md`, `02-architecture.md`) were not touched.

| Page | Words | Contents |
|---|---|---|
| `00-overview.md` | 1,233 | Purpose, what is created, the three flows (traffic, image, infrastructure), what a successful deployment leaves behind, how to read the section |
| `01-project-structure.md` | 1,726 | The tree, a table per file, how Terraform loads a directory, root versus child modules, what could be consolidated and what could not |
| `02-terraform-concepts.md` | 2,243 | Providers, versions, resources and data sources, variables, outputs, locals, functions, `count` versus `for_each`, implicit and explicit dependencies, lifecycle (none used), modules (none used) |
| `03-aws-resources.md` | 2,648 | Every resource: network, cluster, node group, the five add-ons, ECR, CI identity, kubeconfig artefacts, the four Helm releases |
| `04-design-decisions.md` | 3,313 | Fourteen comparisons, each with mechanism, trade-offs, cost and the case where the alternative wins |
| `05-state-and-backend.md` | 1,669 | What state is, the backend arguments, locking, init and migration, security, versioning and recovery, drift and `import`/`mv` |
| `06-iam-and-security.md` | 1,882 | The four roles plus two service-linked ones, trust versus permission policies, least-privilege assessment, secrets, network exposure, encryption, ranked risks |
| `07-deployment-lifecycle.md` | 1,492 | Each command and what it does internally, reading plans, safe updates, destroy ordering, environments |
| `08-ci-cd.md` | 1,310 | The real ECR workflow and OIDC flow, the three-way split between Terraform, CI and Argo CD, and an explicitly illustrative Terraform pipeline |
| `09-troubleshooting.md` | 1,746 | Nine failure classes with symptom, cause, diagnosis, fix and prevention |
| `10-cost-and-production.md` | 1,834 | Billing drivers, idle costs, optimisation options, HA analysis, scaling, monitoring, backup, implemented versus recommended, a fix order |

Grounding: every page was written from the files in `terraform/` and the workflow in `banking-system/.github/workflows/ci-ecr.yml`, which were read in full before writing. The pages state plainly that the stack has been validated but not applied.

Two codebase corrections came out of writing it:

- `ci-ecr.yml` still set `AWS_REGION: us-east-1` after the project moved to `us-west-1`, which would have failed the ECR login and push. Corrected.
- A design-decision section on the local ClusterIssuer chart was missing even though the resources page linked to it. Added.

Note: `08-ci-cd.md` records that no Terraform pipeline exists and marks the sample workflow as illustrative rather than implemented, so the documentation does not claim automation the project does not have.


### Entry 20 —  — Kubernetes documentation section

Source: instruction from the project owner, asking for a comprehensive Notion-style section on the Kubernetes manifests.

Deliverable: a new top-level section at `documentation/kubernetes/`, fourteen pages, about 24,600 words, all internal links verified. The earlier single page is untouched apart from one cross-reference line.

| Page | Words | Contents |
|---|---|---|
| `00-overview.md` | 1,223 | Purpose, architecture diagram, traffic flow, change flow, expected outcome |
| `01-project-structure.md` | 1,438 | Every file, apply order, what `kubectl apply` does, namespaces, consolidation options, placeholders |
| `02-cluster-and-concepts.md` | 2,420 | Control plane and node components, reconciliation in seven steps, every object kind, labels, requests, probes, init containers, what is absent |
| `03-workloads.md` | 2,124 | The two Deployments and the StatefulSet field by field, failure consequences, scheduling, alternatives |
| `04-networking.md` | 2,209 | VPC CNI, three Service types, endpoints, DNS, kube-proxy versus load balancing, Ingress, TLS, the ten-step request path, missing NetworkPolicy |
| `05-configuration-and-secrets.md` | 1,547 | ConfigMap, Secret, `envFrom` versus `valueFrom`, rotation, base64 versus encryption, external secret alternatives |
| `06-storage.md` | 1,703 | `emptyDir`, StorageClass fields, dynamic provisioning, lifecycle table, access modes, backups |
| `07-security.md` | 1,597 | Authentication versus authorisation, securityContext per workload, Pod Security Standards, an attacker walkthrough, ranked recommendations |
| `08-deployment-lifecycle.md` | 1,741 | Every command and its internal effect, rollout mechanics, rollback, scaling, deletion |
| `09-containers-and-images.md` | 1,510 | Both Dockerfiles, why `npm ci`, unprivileged nginx, ECR, pull policy, commit-to-pod path |
| `10-helm.md` | 1,224 | What is Helm and what is not, the local chart's template syntax, Helm versus plain YAML versus Kustomize |
| `11-ci-cd-and-gitops.md` | 1,658 | The image pipeline, Argo CD architecture, Application fields, prune and selfHeal, sync versus health, what is absent |
| `12-troubleshooting.md` | 2,553 | Fourteen failure classes with meaning, causes, diagnosis, fix and prevention |
| `13-availability-scaling-cost.md` | 1,693 | What protects availability, failure analysis, scaling layers, upgrades, cost drivers, implemented versus recommended |

Grounding: every page was written from the twelve manifests, the Argo CD ingress, both Dockerfiles, the nginx config and the CI workflow, all read in full first. The section states that the manifests have not been applied to the cluster.

Note: the earlier `documentation/01-kubernetes-deployment.md` now carries a single line pointing to the new section, so the two do not contradict each other. Nothing else in it was changed.
