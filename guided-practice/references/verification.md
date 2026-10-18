# Verifying the user's work safely

The rule: **evidence before verdict, and safety before evidence.** Use the strongest check that can't hurt anything. If the only strong check is risky, explain it and let the user run it.

## Contents
1. Universal first checks
2. Code (functions, modules)
3. Web front-ends
4. Shell, environment and config files
5. Git
6. Databases and SQL
7. Infrastructure and cloud
8. Data files and spreadsheets
9. When you can't verify

---

## 1. Universal first checks
- **Is it saved?** A 0-byte or unchanged file right after "done" usually means unsaved changes in the editor. Suggest autosave.
- **Right place, right name?** Folder names, case sensitivity (`Painel.jsx` vs `painel.jsx`) and extensions.
- **Read the whole thing.** Don't skim for the part you expected. Users change things you didn't ask for, and sometimes that's the bug.

## 2. Code
- **Parse and lint** (`eslint`, `ruff`, `tsc --noEmit`, `go vet`...). Know the blind spots: linters don't check that import paths exist, that names match across files, or that logic is right.
- **Execute with their real data** against the expected-results table you gave before they started. Add one or two edge cases: an empty list, a missing id, the boundary value.
- Check that **names and signatures match** what other files (or teammates) will import.
- For **recursion, loops and async code**: watch for infinite recursion (a function calling itself as its own answer), unawaited promises, and mutation of inputs.
- Prefer a throwaway test script in a scratch location over adding files to their project. Delete or ignore it afterwards.

## 3. Web front-ends
- **Build** (`npm run build` or equivalent) to catch unresolved imports and syntax errors in files that aren't open.
- **Load each route** and confirm the expected content. Test a hard refresh (F5) on deep links.
- Check the **browser console** for errors.
- For **responsive layout**, check a narrow width.

## 4. Shell, environment and config files
- Prefer **read-only commands** to confirm state: `pwd`, `ls`, `which`, `--version`, `cat`, `env | grep`.
- **Parse config files** before relying on them (`jq . file.json`, `yamllint`, `python -c "import tomllib..."`).
- For anything that **changes the system** (installs, permissions, profiles, services), explain it and let the user run it.

## 5. Git
- Read-only commands are always safe: `git status`, `git log --oneline`, `git diff`, `git show --stat`, `git ls-files`, `git fetch` then `git status -sb`.
- **Check the commit contents, not just the message**: did `node_modules`, secrets or OS junk sneak in?
- **Never push, force-push, rebase shared branches, delete branches or rewrite history on your own.** Explain it, and let the user decide.

## 6. Databases and SQL
- Start with **read-only queries**, and use `LIMIT` on exploration.
- Use **`EXPLAIN`** (not `EXPLAIN ANALYZE` on production, which runs the query) to check plans.
- **Writes**: run them in a transaction and check the affected row count before `COMMIT`. On anything shared, write `BEGIN; ... ; ROLLBACK;` first.
- **Migrations**: review the generated SQL, check reversibility, and run on a local or staging copy first.
- **Never run destructive statements** (`DELETE` or `UPDATE` without a `WHERE`, `DROP`, `TRUNCATE`) against shared data yourself.

## 7. Infrastructure and cloud
- Use **plan, diff and dry-run modes**:
  - `terraform plan`, `terraform validate`, `terraform fmt -check`;
  - `kubectl diff`, `kubectl apply --dry-run=server`;
  - `helm template`, `helm lint`;
  - `aws ... --dry-run` where supported.
- **Read the plan with the user**: what is created, changed or destroyed. Treat any destroy or replace as a red flag to discuss.
- **`apply`, deploys and credential changes are the user's to run**, after review, and only if their team's process allows it.

## 8. Data files and spreadsheets
- **Parse the file** and check row counts, column names, types and nulls.
- **Spot-check a few rows** by hand against the expected result.
- For **formulas**, recompute two or three cells independently.

## 9. When you can't verify
Say so plainly, and give the user the exact command or click-path to check it themselves, plus what a good and a bad result look like. An honest "I couldn't run this; here's how to check" beats a confident guess.
