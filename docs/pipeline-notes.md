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
