# Git workflow

Branches, commit messages and pull requests for this repository.

## Branches

| Branch | Purpose | Rules |
| --- | --- | --- |
| `master` | Production. Render deploys it. | Receives only release PRs from `develop` and hotfixes. |
| `develop` | Integration branch. Always working. | Receives work branches through pull requests. |
| `<type>/<short-name>` | One story, fix or doc change. | Cut from `develop`. Deleted after merge. |

Branch names use the commit type as prefix, lowercase, words separated by `-`:
`feat/question-list`, `fix/login-cookie`, `docs/git-workflow`, `chore/update-deps`.

## Daily flow

```bash
git switch develop
git pull
git switch -c feat/<short-name>
# ... work, commit (Conventional Commits) ...
git push -u origin feat/<short-name>
```

1. Open a pull request into **`develop`**. The PR template is filled in automatically.
2. Title = Conventional Commit (checked by the `PR title` workflow).
3. `npm run verify` must pass.
4. **Squash and merge**: the PR title becomes the single commit on `develop`.
5. Delete the branch, then `git switch develop && git pull`.

## Release

Open a pull request **`develop` → `master`** titled `chore(release): <date or version>` and use **Create a merge commit** (not squash), so `develop` and `master` keep a common history. Render deploys `master` after the merge.

## Hotfix

Branch `fix/<short-name>` from `master`, open a PR into `master`, merge, then merge `master` back into `develop`.

## Commit messages (Conventional Commits)

```
type(scope): subject

optional body: what and why, not how

optional footer: BREAKING CHANGE: ..., Refs: docs/features/<name>.md
```

**Types**

| Type | Use for |
| --- | --- |
| `feat` | New user-visible behaviour |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `refactor` | Code change that neither fixes a bug nor adds behaviour |
| `test` | Adding or fixing tests |
| `perf` | Performance improvement |
| `build` | Build system, dependencies |
| `ci` | GitHub Actions and other CI config |
| `chore` | Maintenance that fits nowhere else (including releases) |
| `style` | Formatting only |
| `revert` | Reverts an earlier commit |

**Scopes** (optional): `web`, `api`, `e2e`, `docs`, `db`, `deps`.

**Subject rules:** imperative mood ("add", not "added"), lowercase start, no trailing period, at most 72 characters. Use `!` after the type or scope, or a `BREAKING CHANGE:` footer, for incompatible changes.

Examples:

```
feat(api): add register and login endpoints
fix(web): show empty state when a topic has no questions
docs: record hosting decision
test(api): cover status decay after 14 days
chore(release): 2026-12-01
```

## Pull request descriptions

The template in `.github/pull_request_template.md` is applied to every new PR. Fill in Summary, Changes, Related, How to test and the Checks list. When an AI agent opens the PR it writes the description from the commits and the diff, following the same template.

Agents do not commit or push unless asked (see `AGENTS.md`).
