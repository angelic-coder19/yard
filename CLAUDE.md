# Yard: how Claude works in this repo

## Who I am and what this repo is for
I'm Angel, a CS sophomore at UAPB. Yard is an HBCU campus event aggregator: it collects events from where they already live (Instagram, campus calendars, GroupMe, Facebook) and puts them in one feed per campus. Yard is three things at once: my Software Engineering class project (full SDLC, Phase 1 planning package already done), my HBCUFi pitch, and my capstone.

My real goal is not just a working app. I want to be able to explain every line, defend every decision against alternatives, and talk about it in an Amazon SDE (AWS Database) intern interview. A past project taught me what happens when Claude builds things I can't explain. That doesn't happen again.

**The rule: if I can't explain a line, it doesn't merge.**

## Hard rules for Claude
1. **One commit at a time.** Never work ahead. Finish the current commit, then stop and wait for me.
2. **Never run `git commit`, `git push`, or `git merge`.** I do those myself. You may run `git status`, `git diff`, and `git log` to show me things.
3. **Never write my commit message.** I write it in my own words. That's my homework. After I write it, critique it: is the subject imperative and under ~50 chars, does the body explain *why*, does it match the diff, is anything missing or wrong? Then I rewrite it if needed.
4. **Respect the ownership tiers below.** For "Angel types" files, give me the plan, a skeleton with TODOs, or hints. Never the finished code. If I'm stuck after a real attempt, give the smallest hint that unblocks me.
5. **Teach small.** Each concept gets 3 to 6 sentences max, tied to Yard specifically. No textbook dumps.
6. **Ask before you assume.** If a step depends on a choice, show me 2 to 3 options with tradeoffs in one or two lines each, recommend one, and let me pick.
7. **Security is part of Done.** No secrets in code or git, validate all input, no sensitive data in errors or logs.

## Ownership tiers
- **Angel types it (Claude reviews only):** SQL schema and migrations, the feed query, indexes, the cache layer (`getFeed` + invalidation), DynamoDB access code, input validation on write routes.
- **Angel leads, Claude fills:** route handlers, the data access layer, tests for the core. I write pseudocode or a skeleton first, you complete it, I explain it back.
- **Claude writes, Angel skims:** UI layout and styling, config/scaffolding, seed scripts, docker-compose. Point out anything that touches data or secrets.

## The loop for every commit
Follow this exact order. Label each step so I know where we are.

1. **Branch.** Tell me the branch name (`feat/…`, `chore/…`, `perf/…`, `test/…`). I create it.
2. **Goal (2 sentences).** What this commit adds and why Yard needs it now.
3. **Concept (short).** The one or two ideas I need to understand. Use Yard examples.
4. **Decision.** What we're choosing, plus 2 to 3 alternatives and why we're not using them. If it's a real architectural choice, I write an ADR in `docs/decisions/NNNN-title.md` (Context / Decision / Alternatives / Consequences).
5. **Build.** In the order the ownership tier says. Keep the diff small: one idea per commit.
6. **Run it.** Show me how to prove it works (a command, a curl, a query, a test). I run it.
7. **Explain-back gate.** Ask me these 3 questions and wait for my answers. If I'm wrong or vague, correct me and ask again:
   - What does this change do, in one sentence?
   - Why this approach over one specific alternative?
   - What breaks if I delete it?
8. **Build Log drill.** Ask me 1 or 2 interview-style questions from the list below that this commit prepares me for. Grade my answer honestly and show me a stronger version in engineer language.
9. **Git skill of the commit.** Teach the one git skill listed for this commit, and have me use it.
10. **I stage, I write the message, you critique, I commit, I push, I open a PR to `main` and merge it.** Then stop.

## Commit plan (tonight + Friday)
Deadline is Sunday. Saturday is buffer only. Do not add scope. If we fall behind, cut from the bottom, never skip the explain-back.

### Tonight (Thursday)
| # | Branch | What | Tier | Git skill |
|---|---|---|---|---|
| 0 | `chore/repo-hygiene` | .gitignore + ADR 0001 filled in | **Angel** | Stage specific files, not `git add .` |J
| 1 | `chore/claude-context` | Add this CLAUDE.md | Angel | Feature branchJ`` + PR flow, even solo |
| 2 | `chore/scaffold-next` | Next.js (TypeScript, App Router) scaffolded into the existing repo, keeping README, docs/, and merging .gitignore. `create-next-app` won't run in a non-empty folder with its own git, so scaffold to a temp dir with `--disable-git` and move files in. Walk me through every top-level file it created. | Claude | `git status` vs `git diff --staged`; reading a big diff with `git diff --stat` |
| 3 | `chore/local-infra` | `docker-compose.yml` with Postgres and Redis, `.env.example` (committed) vs `.env.local` (ignored), a `db` client module using `pg` | Claude writes, Angel explains ports/volumes/env | `git add -p` (stage hunks, not files) |
| 4 | `feat/db-schema` | Raw SQL migration: schools, organizations, events, users, rsvps. Foreign keys, `UNIQUE (user_id, event_id)` on rsvps, timestamps. A tiny script to run migrations. | **Angel types the SQL** | `git commit --amend` (fix the last commit before pushing) |

### Friday
| # | Branch | What | Tier | Git skill |
|---|---|---|---|---|
| 5 | `chore/seed` | Seed ~50,000 events across 10 schools | Claude | `.gitignore` check: prove no `.env` is tracked with `git ls-files` |
| 6 | `feat/feed-api` | Data layer `feedFor(schoolId, limit)` + `GET /api/schools/[schoolId]/feed`. Validate params. Check for N+1. | Angel leads (query is Angel's) | `git log --oneline --graph` and `git show <sha>` |
| 7 | `perf/feed-index` | `EXPLAIN ANALYZE` before, add `(school_id, starts_at)` index, after. Save both plans + timings in `docs/perf/feed-index.md`. | **Angel types** | `git diff main..HEAD` (compare branches) |
| 8 | `feat/feed-ui` | Feed page for one school using the API | Claude | `git stash` (park work, switch branches, come back) |
| 9 | `feat/feed-cache` | Redis cache-aside in `getFeed`: versioned key `feed:v1:{schoolId}`, 5-min TTL + jitter, fall through to Postgres if Redis is down. ADR 0002. | **Angel types** | `git rebase -i` to squash my WIP commits before the PR |
| 10 | `feat/event-write` | `POST/PATCH /api/events` with input validation (zod), delete cache key after DB write, test proving an edited room shows up immediately | Angel types validation + invalidation; Claude fills route | Squash-merge vs merge commit on the PR, and why |
| 11 | `test/load` | k6 script; record p50/p95 cache off vs on and hit ratio in `docs/perf/feed-cache.md`. Real numbers only. | Angel leads | `git tag -a v0.1.0` and push the tag |

**Stretch (only if 1 to 11 are done and I can explain all of them):** `feat/ingest-dynamo`. DynamoDB Local in docker-compose, an `ingest` table designed from access patterns (write the access-pattern table first), conditional put with `attribute_not_exists(PK)` on a content hash, proven by sending the same item twice. ADR 0003. No real AWS account needed for this.

## Build Log questions I must be able to answer by Sunday
Use these for step 8. Map each commit to the ones it prepares me for.
- Walk me through what happens when a student opens Yard. (2, 6, 9)
- Why didn't you just use NoSQL for everything? (4, stretch)
- Feed query is `WHERE school_id = $1 AND starts_at > now() ORDER BY starts_at LIMIT 20`. What index and why? (7)
- What is the N+1 query problem and did you have one? (6)
- Why does Postgres struggle when hundreds of serverless functions connect at once? (3)
- How do you keep the cache and the database consistent? (9, 10)
- What's a cache stampede and how would you prevent it? (9)
- Redis goes down. What do students see? (9)
- Why delete the cache key on write instead of overwriting it? (10)
- What does p95 mean and why not the average? (11)
- How does an index make a lookup fast? (7)
- What should never appear in logs or git? (3, 5, 10)
- SQL vs NoSQL: how do you choose? (4, stretch)
- What happens if the same message is processed twice? (stretch)

## Vocabulary to use on me
Use these terms precisely when you explain, and make me use them back: source of truth, access pattern, idempotent, hot path / cold path, staleness budget, p95 latency, hit ratio, blast radius, cache-aside, invalidation, composite index, sequential scan vs index scan, N+1, connection pool, least privilege.

## Style
- Direct and brief. No filler, no motivational talk.
- Honest. If my answer is weak, say so and show the strong version.
- When I say "skip the lesson," just do the step, but still run the explain-back gate.