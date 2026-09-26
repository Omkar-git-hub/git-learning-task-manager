# Phase 7 — Wrong Commit & Recovery Decision Guide

> A practical guide for choosing between `git restore`, `git reset`, `git revert`, `git commit --amend`, and `git reflog`.

---

## 1. The Main Decision

The first question is:

**What went wrong, and has it been committed or shared yet?**

```text
WRONG CHANGE
     │
     ├── NOT committed
     │      ├── Discard file changes
     │      │      → git restore <file>
     │      │
     │      └── Accidentally staged a file
     │             → git restore --staged <file>
     │
     ├── Latest commit needs fixing
     │      → git commit --amend
     │
     ├── Remove local commits
     │      → git reset
     │
     ├── Commit already pushed/shared
     │      → git revert
     │
     └── Commit appears lost
            → git reflog
            → git show <hash>
            → git branch recovery <hash>
```

---

# 2. `git restore`

## Purpose

`git restore` is mainly used to **restore file content**.

It does **not move `HEAD`** and does **not remove commits**.

### Discard changes in a file

```bash
git restore <file>
```

Use when:

- You modified a file.
- You have **not committed** the change.
- You want to discard the modification.

Example:

```text
You changed app.py
You did NOT commit it.
```

```bash
git restore app.py
```

Result:

```text
app.py → restored to the current HEAD version
```

---

## Unstage a file

```bash
git restore --staged <file>
```

Use when:

- The file is staged.
- You want to remove it from the Staging Area.
- You want to keep the actual file changes.

Example:

```bash
git add app.py
git restore --staged app.py
```

Result:

```text
Staging Area       → app.py removed
Working Directory  → changes remain
```

### Mental model

> **`restore` = Fix/restore my FILE.**

---

# 3. `git reset`

## Purpose

`git reset` is used to **move the current branch/`HEAD` to another commit**.

Example:

```text
A → B → C
```

```bash
git reset HEAD~1
```

Result:

```text
A → B
```

The exact effect on your files depends on the reset mode.

### Important

`git reset` changes **local history**.

It is generally appropriate when working with history that has not been shared.

---

# 4. Reset Modes

There are three important modes:

| Mode | HEAD | Staging Area | Working Directory |
|---|---|---|---|
| `--soft` | Moves | Keeps changes staged | Keeps changes |
| `--mixed` | Moves | Unstages changes | Keeps changes |
| `--hard` | Moves | Resets | Resets |

---

## 4.1 Soft Reset

```bash
git reset --soft <commit>
```

Example:

```text
A → B → C
```

```bash
git reset --soft B
```

Result:

```text
A → B
```

Changes introduced by `C` remain **staged**.

### Use when

You want to:

- Remove one or more commits.
- Keep their changes.
- Immediately create a new commit.

### Mental model

> **SOFT = Move HEAD + Keep changes STAGED**

---

## 4.2 Mixed Reset

```bash
git reset --mixed <commit>
```

or simply:

```bash
git reset <commit>
```

`--mixed` is the default reset mode.

Example:

```text
A → B → C
```

```bash
git reset --mixed B
```

Result:

```text
A → B
```

Changes introduced by `C` remain in the **Working Directory**, but they are **unstaged**.

### Use when

You want to:

- Remove commits.
- Keep the file changes.
- Review/edit them before staging again.

### Mental model

> **MIXED = Move HEAD + Keep files + Unstage**

---

## 4.3 Hard Reset

```bash
git reset --hard <commit>
```

Example:

```text
A → B → C
```

```bash
git reset --hard B
```

Result:

```text
A → B
```

Changes introduced by `C` are removed from the current branch state and Working Directory.

### Use when

You intentionally want to:

- Throw away local changes.
- Return your Working Directory to an earlier commit.

### ⚠️ Warning

Uncommitted changes can be permanently lost.

Before using `--hard`, make sure you really want to discard the changes.

### Mental model

> **HARD = Move HEAD + Reset Staging + Reset Files**

---

# 5. `git revert`

## Purpose

`git revert` **undoes a commit by creating a new commit**.

It does not remove the original commit.

Example:

```text
A → B → C
```

```bash
git revert C
```

Result:

```text
A → B → C → D
```

Where:

```text
D = new commit that reverses C
```

### Use when

- The commit has already been pushed.
- Other developers may already have it.
- You want to preserve the existing history.

### Important

`revert` does **not rewrite history**.

### Mental model

> **REVERT = Undo a commit with another commit.**

---

# 6. `git commit --amend`

## Purpose

`git commit --amend` modifies the **latest commit**.

Useful when:

- You forgot a file.
- You want to fix the latest commit message.
- You need a small correction in the latest commit.

### Fix the latest commit message

```bash
git commit --amend -m "New commit message"
```

### Add a forgotten file

```bash
git add README.md
git commit --amend
```

Example:

```text
A → B
```

`B` is the latest commit.

After amend:

```text
A → B'
```

`B'` is a new commit replacing `B`.

### Important

Amend changes the commit ID.

Avoid amending commits that have already been shared with other developers.

### Mental model

> **AMEND = Fix my LAST COMMIT.**

---

# 7. `git reflog`

## Purpose

`git reflog` helps you find **previous positions of `HEAD` and local references**.

It is especially useful after:

- `git reset`
- `git rebase`
- accidental checkout/switch operations
- deleting/moving a branch
- other local history changes

Command:

```bash
git reflog
```

---

## `git log` vs `git reflog`

| Command | Shows |
|---|---|
| `git log` | Normal reachable commit history |
| `git reflog` | Previous local `HEAD`/reference positions |

### Mental model

> **`git log` = Where is my history now?**

> **`git reflog` = Where has my `HEAD` been?**

---

# 8. Recovering a Lost Commit

Suppose you have:

```text
A → B → C
```

Then accidentally run:

```bash
git reset --hard A
```

Now your current branch points to:

```text
A
```

You may no longer see `B` and `C` in the normal branch history.

## Step 1 — Find the commit

```bash
git reflog
```

## Step 2 — Inspect it

```bash
git show <hash>
```

## Step 3 — Safely protect it

```bash
git branch recovery <hash>
```

Now:

```text
A ← main

C ← recovery
```

Your recovered commit is safely referenced by the `recovery` branch.

### Mental model

```text
reflog
   ↓
FIND

git show
   ↓
CHECK

git branch recovery
   ↓
SAVE
```

---

# 9. `restore` vs `reset` vs `revert`

| Command | Main purpose | Changes commit history? |
|---|---|---|
| `git restore` | Restore file content | ❌ No |
| `git reset` | Move branch/`HEAD` backward | ✅ Yes |
| `git revert` | Undo a commit with a new commit | ❌ No |
| `git commit --amend` | Modify latest commit | ✅ Yes |
| `git reflog` | Find previous `HEAD` positions | ❌ No |

---

# 10. Simple Mental Model

```text
RESTORE
"Fix/restore my FILE."

RESET
"Move my BRANCH BACK."

REVERT
"UNDO a COMMIT with another COMMIT."

AMEND
"Fix my LAST COMMIT."

REFLOG
"Find where my HEAD WAS."
```

---

# 11. Wrong Commit Decision Guide

## Case A — Not committed yet

### Want to discard file changes?

```bash
git restore <file>
```

### Accidentally staged a file?

```bash
git restore --staged <file>
```

---

## Case B — Latest commit needs fixing

```bash
git commit --amend
```

Use for:

- Forgotten file
- Wrong commit message
- Small correction to the latest commit

---

## Case C — Remove local commits

Use:

```bash
git reset
```

Choose the mode based on what you want to keep.

### Keep changes staged

```bash
git reset --soft <commit>
```

### Keep changes but unstage them

```bash
git reset --mixed <commit>
```

### Completely discard changes

```bash
git reset --hard <commit>
```

---

## Case D — Commit is already pushed/shared

Use:

```bash
git revert <commit>
```

This keeps the existing history and adds a new undo commit.

---

## Case E — Commit appears lost

Use:

```bash
git reflog
```

Then:

```bash
git show <hash>
git branch recovery <hash>
```

---

# 12. Real-World Examples

## Case 1 — Edited a file accidentally

You changed:

```text
app.py
```

You did not commit.

Use:

```bash
git restore app.py
```

---

## Case 2 — Accidentally staged a file

You ran:

```bash
git add app.py
```

But you don't want it staged.

Use:

```bash
git restore --staged app.py
```

Your changes remain.

---

## Case 3 — Bad local commit, keep changes staged

History:

```text
A → B → C
```

`C` is wrong and nobody has received it.

Use:

```bash
git reset --soft HEAD~1
```

Result:

```text
A → B
```

The changes from `C` remain staged.

---

## Case 4 — Bad local commit, keep changes unstaged

History:

```text
A → B → C
```

Use:

```bash
git reset --mixed HEAD~1
```

Result:

```text
A → B
```

The changes from `C` remain in the Working Directory but are unstaged.

---

## Case 5 — Completely throw away a local commit

History:

```text
A → B → C
```

You don't need anything from `C`.

Use:

```bash
git reset --hard B
```

⚠️ Make sure you really want to discard the changes.

---

## Case 6 — Bad commit was already pushed

History:

```text
A → B → C
```

`C` is wrong and already shared.

Use:

```bash
git revert C
```

Result:

```text
A → B → C → D
```

`D` reverses the changes from `C`.

---

## Case 7 — Forgot a file in the latest commit

History:

```text
A → B
```

`B` is the latest commit and `README.md` was forgotten.

Use:

```bash
git add README.md
git commit --amend
```

The latest commit is replaced by a corrected commit.

---

## Case 8 — Accidentally reset too far

History:

```text
A → B → C
```

You run:

```bash
git reset --hard A
```

You now need `B` or `C`.

First:

```bash
git reflog
```

Find the required hash.

Then:

```bash
git branch recovery <hash>
```

Now the commit is safely referenced by the recovery branch.

---

# 13. Quick Decision Table

| Situation | Command |
|---|---|
| Discard uncommitted file changes | `git restore <file>` |
| Unstage a file | `git restore --staged <file>` |
| Fix latest commit | `git commit --amend` |
| Remove local commits, keep staged changes | `git reset --soft` |
| Remove local commits, keep unstaged changes | `git reset --mixed` |
| Remove local commits and discard changes | `git reset --hard` |
| Undo a pushed/shared commit | `git revert` |
| Find a commit after accidental reset/rebase | `git reflog` |
| Inspect recovered commit | `git show <hash>` |
| Safely protect recovered commit | `git branch recovery <hash>` |

---

# 14. One Rule to Remember

```text
NOT COMMITTED?
    → restore

WRONG LATEST COMMIT?
    → amend

REMOVE LOCAL COMMITS?
    → reset

UNDO SHARED/PUSHED COMMIT?
    → revert

LOST COMMIT?
    → reflog
```

---

# 15. Safety Rules

Before using reset:

```bash
git status
git log --oneline --graph
```

Before using:

```bash
git reset --hard
```

make sure you really want to discard the changes.

If a commit is already shared with other developers:

```text
Prefer git revert
```

instead of rewriting shared history.

If you accidentally reset or rebase:

```text
Use git reflog first.
```

If you are unsure what you found:

```bash
git branch recovery <hash>
```

Create the recovery branch first, then investigate.

---

## Final Mental Model

```text
┌──────────────────────────────────────────────┐
│              WHAT WENT WRONG?                │
└──────────────────────────────────────────────┘
                      │
        ┌─────────────┴─────────────┐
        │                           │
   NOT COMMITTED                 COMMITTED
        │                           │
     restore                 ┌─────┴─────┐
                             │           │
                         LATEST?     ALREADY SHARED?
                             │           │
                           amend       revert
                             │
                      REMOVE LOCAL COMMITS?
                             │
                           reset
                             │
                    LOST AFTER RESET/REBASE?
                             │
                           reflog
```

> **Core rule:**
>
> **Restore files. Reset local history. Revert shared history. Amend the latest commit. Reflog finds lost history.**
