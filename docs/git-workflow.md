# My E-commerce: Git Workflow

## 1. Branching Strategy (GitHub Flow)

| Branch | Purpose | Rules |
|---|---|---|
| `main` | Always stable and working | Never commit directly; only changes arrive through a Pull Request |
| `feature/<name>` | One new feature | Created from `main`, deleted after merge |
| `fix/<name>` | One bug fix | Created from `main`, deleted after merge |
| `docs/<name>` | Documentation-only change | Created from `main`, deleted after merge |

Examples: `feature/user-register`, `feature/user-login`, `fix/duplicate-email-check`, `docs/git-workflow`.

## 2. Workflow for Every Task

1. Create a GitHub Issue for the task (for example, "Create register API (US-01)").
2. Update `main` locally: `git checkout main` then `git pull`.
3. Create a branch: `git checkout -b feature/user-register`.
4. Write code and make small, frequent commits.
5. Push the branch: `git push -u origin feature/user-register`.
6. Open a Pull Request into `main`. Write `Closes #<issue number>` in the description.
7. Review your own changes in the "Files changed" tab, then merge.
8. Delete the branch, and pull `main` again locally.

## 3. Commit Message Convention

Format: `type: short description (US-xx)`

| Type | Use for |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation only |
| `test` | Adding or changing tests |
| `refactor` | Code change that does not change behavior |
| `chore` | Build files, configuration, dependencies |

Examples:
- `feat: add register API (US-01)`
- `fix: return 409 for duplicate email (US-01)`
- `test: add unit tests for login service (US-02)`
- `docs: add api design`

Rules: write in the present tense, keep the first line under 70 characters, and make one commit do one logical thing.

## 4. Never Commit Secrets

These must never be pushed to GitHub:
- Database passwords
- The JWT secret key
- Cloud access keys

Use environment variables in `application.properties`, for example:

```properties
spring.datasource.password=${DB_PASSWORD}
jwt.secret=${JWT_SECRET}
```

Anything that was committed once stays in the Git history, even if deleted later. If a secret is ever pushed, change it immediately.

## 5. Releases and Tags

At the end of each sprint, when `main` is working, create a tag:

| Tag | Meaning |
|---|---|
| `v0.1.0` | Sprint 1 complete (user-service) |
| `v0.2.0` | Sprint 2 complete (product-service) |
| `v0.3.0` | Sprint 3 complete (order-service) |
| `v0.4.0` | Sprint 4 complete (api-gateway) |
| `v1.0.0` | Sprint 5 complete (deployed) |

Each tag is published as a GitHub Release with short release notes, and `CHANGELOG.md` is updated.

## 6. GitHub Issues Setup

**Labels:** `feature`, `bug`, `docs`, `test`, and one per service (`user-service`, `product-service`, `order-service`, `api-gateway`).

**Milestones:** one per sprint (Sprint 1 to Sprint 5). Each issue is assigned to its sprint's milestone, so the milestone page shows sprint progress.
