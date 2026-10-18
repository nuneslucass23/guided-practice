# Step and review examples

Examples of the same skill at three depths, plus common mistakes. Use them to calibrate. Don't copy them literally.

## Contents
1. Beginner step (terminal command)
2. Beginner step (code with blanks)
3. Intermediate step
4. Advanced step
5. Reviews at each level
6. Handling "do it for me"
7. Common mistakes

---

## 1. Beginner step: terminal command

> ## Step 3: Install the project's libraries
>
> **What this step does:** `package.json` only *lists* the libraries the project uses. This downloads them into a folder called `node_modules`. Without it, the project can't run.
>
> ```bash
> npm install
> ```
>
> **What each part means:**
> - `npm`: Node's package manager. It downloads and runs JavaScript tools.
> - `install`: reads `package.json` and downloads everything listed under `dependencies` and `devDependencies`.
>
> **What you'll see:** 30–60 seconds of output ending in something like `added 150 packages`. Yellow `warn` lines are normal. Red `ERR!` is not; paste it to me if you see one. Two new things appear in the folder: `node_modules/` (huge, never committed) and `package-lock.json` (records exact versions, does get committed).
>
> Tell me when it finishes and I'll check.

Why it works: purpose first, every token explained, the output predicted (including the scary-looking warnings), and it ends with a clear handoff.

## 2. Beginner step: code with blanks

> ## Step 7: Write `isAvailable`
>
> **What this function does:** answers "is this technician free?" It gets a `technicianId` and the list of `jobs`, and returns `true` or `false`.
>
> **The rule:** a technician is busy if some job is **theirs** *and* **in progress** (`'dispatched'` or `'on_the_way'`).
>
> Fill in the `___`:
> ```js
> const inProgress = jobs.filter((j) =>
>   j.technicianId === ___ && (j.status === ___ || j.status === ___)
> )
> return inProgress.length === 0
> ```
> - The first blank: what must `j.technicianId` equal? The technician the function *received*.
> - The other two: the two statuses that make someone busy, in quotes.
> - The parentheses around the `||` part matter. Without them, `&&` is evaluated first and *any* job on the way would count for everyone.
>
> **How I'll test it:**
>
> | Call | Expected | Why |
> |---|---|---|
> | `isAvailable('T1', jobs)` | `true` | T1 has nothing in progress |
> | `isAvailable('T3', jobs)` | `false` | T3 is `on_the_way` on job 6 |

## 3. Intermediate step

> ## Step 4: Add the retry wrapper
>
> **What this does:** the payment client fails on transient 503s. We wrap calls in a retry with exponential backoff, so a blip doesn't become a failed checkout.
>
> Write `withRetry(fn, { attempts, baseMs })` in `lib/retry.ts`. Requirements:
> - retries only on errors where `isTransient(err)` is true (it already exists in `lib/errors.ts`);
> - waits `baseMs * 2 ** attempt` between tries, with jitter;
> - rethrows the last error after the final attempt.
>
> Non-obvious bits:
> - **Jitter** (`Math.random()` on the delay) stops many clients from retrying in lockstep.
> - **Don't retry non-transient errors.** Retrying a 400 just triples the bad request.
>
> **I'll check:** that it retries 503s and stops at `attempts`, that it doesn't retry a 400, and that the delays grow.

Why it works: no syntax lecture, the *why* of each requirement, and the acceptance criteria up front.

## 4. Advanced step

> ## Stage 2: Idempotency for the order consumer
>
> **Decision to make:** the consumer gets at-least-once delivery, so duplicates will happen. Options:
> 1. **Dedup table keyed by message id**, inserted in the same transaction as the side effect. Simple and correct, but adds a write per message and needs a TTL cleanup job.
> 2. **Natural idempotency**: make the write an upsert on `(order_id, version)`. No extra table, but only works if every side effect can be expressed that way. The email send can't.
> 3. **Broker-side exactly-once**: couples you to the broker's transactional API, and still doesn't cover the external email call.
>
> Given the email side effect, I'd lean toward 1, with an outbox for the email. Your call. Sketch the schema and the transaction boundary, and I'll review it for race windows and failure modes (crash after commit but before ack, rebalance mid-batch).

Why it works: it frames a decision, not a keystroke. Trade-offs and failure modes are the content, and the user owns the design.

## 5. Reviews at each level

**Beginner:**
> 🟡 **Almost.** The logic is right, but the import on line 1 says `'.dados/x.json'`. It's missing the `/` after the dot.
> - `./` means "in this folder", so `./dados/x.json` is `src/dados/x.json`.
> - `.dados` is read as a folder literally named `.dados`, which doesn't exist.
>
> The linter didn't catch it because it doesn't check whether files exist. Fix the three imports and I'll run a real test.

**Intermediate:**
> ✅ **Correct**: retries 503s, stops at 3 and rethrows. Verified with a fake that fails twice, then succeeds.
>
> 🟡 Two optional points:
> - `attempts` counts retries here, but callers will read it as total tries. Rename it to `maxRetries`, or count from 1.
> - The jitter is additive (`delay + rand*100`). Full jitter (`rand * delay`) spreads load better under contention.

**Advanced:**
> ❌ **One race to fix before merging.** The dedup insert and the side effect are in the same transaction, good, but the ack happens before commit (line 58). Crash between them and the message is gone, with no effect applied. Move the ack after commit.
>
> 🟡 The TTL job deletes by `created_at < now() - 7d` without an index on `created_at`. That's a sequential scan on a table that grows by about 2M rows a day.
>
> ✅ The outbox relay is clean, and handles poison messages by parking them. Nice.

## 6. Handling "do it for me"

> Done. I only changed the three imports you had, and left the rest as you wrote it.
>
> | Line | Before | After | Why |
> |---|---|---|---|
> | 1–3 | `'.dados/…'` | `'./dados/…'` | missing `/`, so the path pointed to a folder that doesn't exist |
>
> I ran it: all 4 checks pass. Next up is Step 9, where you'll write the routes.

## 7. Common mistakes

- **Running ahead.** Giving three steps at once means you lose the chance to catch the first mistake before it compounds.
- **Explaining the obvious to experts, or skipping the "obvious" for beginners.** Both come from not calibrating per topic.
- **Judging by eye.** "Looks right!" without running it. Typos in paths, unsaved files and off-by-one errors survive eyeballing.
- **Silent rewrites.** When fixing their code, replacing it with your preferred style, or refactoring things they didn't ask about.
- **Unannounced errors.** Letting them hit an expected error without warning, so they think they broke something.
- **Vague verdicts.** "Mostly fine, a few tweaks." Say which tweaks, whether they are required, and why.
- **Overriding house rules.** Teaching "the right way" when the team or repository has its own way. Theirs wins.
