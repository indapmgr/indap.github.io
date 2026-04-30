# 🌿 Part 7 — Advanced Git

> **Key idea:** `main` must always be stable. All work happens in branches. Code reaches `main` only through pull requests.

---

## Navigation

[← Part 6: Debugging](part-6-debugging) | [Part 8: Full Project →](part-8-full-project)

---

## 1. Real Team Git Workflow

In a real team, `main` is the deployed, stable branch. Nobody pushes directly to it.

**Rules:**

- `main` is always stable and deployable
- Every piece of work gets its own branch
- Branches are merged into `main` through pull requests (PRs)
- PRs require review before merging

---

## 2. Standard Feature Branch Workflow

```bash
# 1. Make sure you are on main and up to date
git checkout main
git pull

# 2. Create a new branch for your task
#    Use a descriptive name: feature/, fix/, chore/
git checkout -b feature/add-user-api

# 3. Do your work (edit files, write code)

# 4. Stage all changes
git add .

# 5. Commit with a clear, descriptive message
git commit -m "Add user API endpoint with validation"

# 6. Push the branch to GitHub
git push -u origin feature/add-user-api
```

Then open a Pull Request on GitHub.

---

## 3. Pull Requests

A pull request (PR) asks the team to review your changes before they are merged into `main`.

**Why PRs exist:**

- Catch bugs before they reach production
- Share knowledge across the team
- Maintain code quality standards
- Track what changed and why (valuable history)

**What is expected in a PR:**

| Expectation | Detail |
|-------------|--------|
| Clear title | Describes what the change does in one line |
| Description | Explains *why* the change was needed and *how* to verify it |
| Small scope | One feature or fix per PR — easier to review |
| Tests pass | CI should be green before requesting review |
| No direct pushes to `main` | Always go through a PR |

---

## 4. Commit Hygiene

| Good commit message | Bad commit message |
|---------------------|-------------------|
| `Add user creation endpoint` | `stuff` |
| `Fix null pointer in user service` | `fix` |
| `Refactor DB connection handling` | `asdfgh` |
| `Add age validation for user creation` | `changes` |

**Format:**

```text
<verb> <what> [<context>]

Examples:
  Add user creation endpoint
  Fix null pointer when user not found
  Refactor DB connection to use context manager
  Remove unused imports from user service
```

> **Tip:** Write commit messages as if completing the sentence: "If applied, this commit will…"

---

## 5. Merge vs Rebase

| Operation | What it does | When to use |
|-----------|-------------|-------------|
| `git merge` | Creates a merge commit combining two branches | Standard team workflow |
| `git rebase` | Replays your commits on top of another branch | Cleaning up local history before a PR |

> **Warning:** Never rebase a branch that has already been pushed and shared with others. It rewrites history and causes conflicts for everyone else.

---

## 6. Resolving Merge Conflicts

Conflicts happen when two branches change the same lines in the same file.

```bash
# Pull main into your feature branch to incorporate the latest changes
git checkout feature/add-user-api
git merge main
```

If there are conflicts, Git marks them in the file:

```text
<<<<<<< HEAD
your version of the code
=======
the incoming version from main
>>>>>>> main
```

Resolution steps:

1. Open the conflicting file(s).
2. Decide what the correct code should be.
3. Remove all conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
4. Save the file.
5. Stage and commit:

```bash
git add <file>
git commit -m "Resolve merge conflict in user service"
```

---

## 7. Undoing Mistakes

### Undo the last commit (keep changes staged, ready to recommit)

```bash
git reset --soft HEAD~1
```

### Discard all unstaged changes (dangerous — changes are lost permanently)

```bash
git checkout .
```

### Unstage all staged files (keep changes on disk)

```bash
git reset
```

### Undo a commit that has already been pushed

Do **not** use `git reset --hard` on pushed commits — it rewrites shared history.

Instead, create a new commit that reverses the previous change:

```bash
git revert HEAD
```

---

## 8. Git States Reference

| State | How you got here | How to move forward |
|-------|-----------------|---------------------|
| Untracked file | New file created | `git add <file>` |
| Modified (unstaged) | Edited a tracked file | `git add <file>` |
| Staged | Ran `git add` | `git commit -m "..."` |
| Committed (local) | Ran `git commit` | `git push` |
| Pushed (remote) | Ran `git push` | Open a PR on GitHub |

---

## 9. Real Team Rules

| Rule | Why |
|------|-----|
| Never push to `main` | `main` is deployed — bad code breaks production |
| One branch per task | Keeps PRs small and easy to review |
| Pull before you branch | Avoids large conflicts later |
| Meaningful commit messages | Makes history readable and searchable |
| Review before merging | Two eyes catch more bugs than one |

---

## Exercises

1. Create a branch called `feature/list-users`, add a `GET /users` endpoint if you haven't already, push it, and open a PR.
2. On a test repo, create two branches that change the same line. Merge one into `main`, then merge the other and resolve the conflict.
3. Use `git reset --soft HEAD~1` to undo a commit, update the message, and recommit.

---

## Part 7 Summary

| Concept | Key Takeaway |
|---------|-------------|
| Branch workflow | Branch → Commit → Push → PR → Review → Merge |
| `main` stability | Never push directly; always use PRs |
| Commit messages | Describe what and why, not how |
| Merge conflicts | Read the markers, choose the correct code, remove markers, commit |
| Undoing mistakes | `--soft` keeps changes staged; `checkout .` discards unstaged changes |

---

## Navigation

[← Part 6: Debugging](part-6-debugging) | [Part 8: Full Project →](part-8-full-project)
