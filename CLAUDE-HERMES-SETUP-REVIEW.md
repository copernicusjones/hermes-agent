# Hermes Agent Setup Review

**Date:** 2026-06-09
**Reviewer:** Claude (cloud session, read-only audit)
**Intended destination:** `/Users/jon/Documents/novaaaaa/20-Reference/AI and Automation/CLAUDE-HERMES-SETUP-REVIEW.md`
(copy this file there — see Scope Notice below)

---

## ⚠️ Scope Notice — read this first

This audit ran in a **cloud Linux container** containing a fresh clone of
`copernicusjones/hermes-agent`, **not on your Mac**. `/Users/jon` does not exist in this
environment. This is itself the exact "wrong-host" failure mode you asked the audit to guard
against, and it was caught before any action was taken.

**What could be verified here (Verified locally = verified in this repo clone):**
- The exact Hermes source version your fork is pinned to (v0.15.1), its git history, your
  local patches, and divergence from your fork's `main`
- Photon and BlueBubbles support in the installed source
- The Discord plugin, allowlist/approval mechanisms, fallback-chain support, cron
  implementation, profile system, `hermes doctor`, and the stock Claude Code skill
- Owl Alpha's status (via web, from OpenRouter's own listing)

**What could NOT be verified and is labeled Unverified throughout:**
- `~/.hermes/config.yaml`, `.env` variable names, live gateway state, launchd services,
  cron jobs actually scheduled, logs, session storage
- Everything in the Obsidian vault (`~/Documents/novaaaaa/...`), all identity files
- Claude Code installation/auth on the Mac
- Whether Saint is currently running, and on what settings

Section H Phase 1 includes a **Mac-local verification checklist** with exact commands so the
unverified half of this audit can be completed in minutes on the right host.

---

## A. Executive verdict

- **Is the setup healthy?** Partially verifiable. The *software base* is healthy: a current,
  capable Hermes v0.15.1 fork with only a one-line functional patch. The *runtime* could not
  be inspected from this session. [Verified locally / Unverified as noted]
- **Is it overcomplicated?** No — and that's the good news. Almost everything you want
  (Discord allowlists, model fallback chains, per-agent profiles, cron, Claude Code
  delegation, health checks) **already exists as a stock Hermes feature**. There is no
  evidence of a custom framework, and you should keep it that way. [Verified locally]
- **What is currently working?** Your fork builds on a recent upstream with profiles,
  a Discord plugin adapter, a maintained Claude Code orchestration skill (v2.2.0), and
  `hermes doctor` for health checks. [Verified locally]
- **What is fragile?** Two things. (1) **Owl Alpha is a stealth model**: free today, logs
  your prompts/completions for training, and will vanish without notice when the lab behind
  it de-cloaks it. If no fallback chain is configured, Saint goes dark the day that happens.
  (2) **Your fork's `main` carries local patches and runtime memory files**, which will make
  every future `git pull` from upstream a merge-conflict event and risks pushing private
  data to GitHub. [Verified locally + Verified from official documentation (OpenRouter)]
- **Biggest risk:** Agent memory files being committed to the GitHub repo.
  `profiles/saint-mac/memories/revenue_ledger.md` (a Gumroad sales ledger) is already
  tracked and pushed. Today it contains only an empty table header — no data has leaked —
  but the *pattern* means the next `git add -A` push publishes Saint's accumulated
  financial/personal memory. [Verified locally]
- **Highest-value improvement:** Configure a `fallback_providers` chain and stop treating
  Owl Alpha as load-bearing. Second: gitignore runtime/profile data so memory never enters
  the repo again. Both are <15-minute changes. [Verified locally — both mechanisms exist
  stock]

---

## B. Current architecture

| Component | Status | Evidence |
|---|---|---|
| **Saint** (primary agent) | Active *(stated by Jon; runtime not observable here)* | `profiles/saint-mac/` exists in repo — Verified locally; live state Unverified |
| **Owl Alpha** (primary model) | Active but **temporary by design** | Free stealth model on OpenRouter, 1M context, logs prompts/completions, no SLA, no announced end date — Verified from OpenRouter listing |
| **Claude Code** | Ready but inactive *(as a Hermes integration)* | Stock skill `skills/autonomous-ai-agents/claude-code/` v2.2.0 present — Verified locally. Mac install/auth — Unverified |
| **Franklin** | Planned | No `profiles/franklin` in repo — Verified locally. Vault-side SOUL — Unverified |
| **Nova** (Windows executor) | Planned / Blocked | Nothing in repo; Windows host not reachable and not attempted per your rules — Verified locally (absence) |
| **Discord** | Active *(stated)*; adapter present | `plugins/platforms/discord/` with `allow_from`/`dm_policy` support — Verified locally. Your actual allowlist config — Unverified |
| **Obsidian vault** | Active *(stated)* | Not reachable from this session — Unverified |
| **Cron** | Feature present; jobs unknown | `cron/scheduler.py`, `cron/jobs.py` (secured dirs, absolute workdir required) — Verified locally. Scheduled jobs — Unverified |
| **Gateway** | Source present; live status unknown | `gateway/` incl. restart handling, shutdown forensics — Verified locally. launchd/run state — Unverified |
| **Photon / iMessage** | **Not present in installed version** | Zero references to "photon" anywhere in code or docs at v0.15.1 — Verified locally |
| **BlueBubbles / iMessage** | Ready but inactive | `gateway/platforms/bluebubbles.py` exists — Verified locally |
| **Monikka / legacy artifacts** | Unknown | Nothing named Monikka in the repo — Verified locally (absence in repo); Mac/vault — Unverified |

---

## C. Findings

### F1 — Audit executed on the wrong host
- **Severity:** Critical (process), self-contained
- **Evidence:** This session runs as root on Linux (`uname`: Linux vm 6.18.5 x86_64); `/Users` does not exist. [Verified locally]
- **Why it matters:** This is precisely the wrong-host failure mode you listed. Any session
  (mine or Saint's) that assumes its host without checking can act on the wrong machine.
  Half of your audit checklist is physically unverifiable from here.
- **Correction:** Re-run the Mac-local checklist (Section H, Phase 1) on the Mac. Longer
  term: add a host-identity check to Saint's operating rules (e.g., refuse host-specific
  actions unless `hostname` matches an expected value).
- **Effort:** 10 min (checklist) / 15 min (host guard note in agent instructions)
- **Jon approval required:** Yes for any instruction-file change (your rule 12).

### F2 — Owl Alpha is a logging, ephemeral stealth model
- **Severity:** High (privacy + reliability)
- **Evidence:** OpenRouter lists `openrouter/owl-alpha` as a $0/$0 stealth model from an
  unnamed provider, 1,048,576-token context, and states **"the model's provider logs prompts
  and completions"** for model improvement. Released 2026-04-28; prior stealth models
  (Polaris Alpha → GPT-5.1, Sherlock Alpha → Grok 4.1) were withdrawn when de-cloaked.
  [Verified from official documentation: openrouter.ai/openrouter/owl-alpha]
- **Why it matters:** (1) Everything Saint sends — vault contents, ideas, eventually message
  traffic — is logged by an unknown party. (2) The model can disappear any day. If no
  fallback chain is configured, Saint stops responding with no obvious cause.
- **Correction:** Keep it for now if cost matters (it is genuinely strong for agentic work),
  but: (a) configure `fallback_providers` in config.yaml — stock feature, see F6; (b) adopt
  a rule that sensitive content (credentials, finances, contacts, private messages) never
  goes through stealth models; (c) treat its disappearance as expected, not an outage.
- **Effort:** 10–15 min config
- **Jon approval required:** Yes (config change).

### F3 — Agent runtime memory committed to the GitHub repo
- **Severity:** High (data exposure pattern; current content harmless)
- **Evidence:** Commit `a4cf603` ("Local patches: auth.py fix + test update + saint-mac
  memory") adds `profiles/saint-mac/memories/revenue_ledger.md` — a "Gumroad Sales Ledger"
  table, currently headers only. `.gitignore` does not exclude `profiles/` or `memories/`.
  [Verified locally]
- **Why it matters:** The moment Saint writes real rows into that ledger and anyone runs
  `git add -A && git push`, personal financial data lands on GitHub. It also violates clean
  vault/runtime separation: agent memory belongs in `~/.hermes` (or the vault), never in the
  source tree.
- **Correction:** Move the file's live home out of the source tree; add `profiles/*/memories/`
  to `.gitignore`; `git rm --cached` the tracked copy. If the repo is private the urgency is
  lower, but the pattern is still wrong.
- **Effort:** 10 min
- **Jon approval required:** Yes (modifies repo and moves a file Saint may be writing to).

### F4 — Fork drift: local patches sit directly on `main`
- **Severity:** Medium
- **Evidence:** Fork `main` = upstream history + one local commit (`a4cf603`) mixing three
  unrelated changes: a one-line `nous-portal`/`nous_portal` provider-alias fix in
  `hermes_cli/auth.py`, a matching test, a 666-line `pnpm-lock.yaml` addition, and the
  memory file from F3. [Verified locally]
- **Why it matters:** Every future upstream pull must merge across this commit. The alias
  fix is upstreamable (it's clean and tested); whether upstream already has it could not be
  checked from this session [Unverified]. The lockfile churn bundled into the same commit
  makes the diff noisy and rebases harder. This is also why "should I update Hermes main?"
  is currently a non-trivial question instead of `git pull`.
- **Correction:** Keep local patches on a branch (e.g., `local-patches`) rebased onto
  upstream, or submit the alias fix upstream and drop the local copy once merged. Split the
  memory file out entirely (F3).
- **Effort:** 20–30 min
- **Jon approval required:** Yes (git history restructuring).

### F5 — Photon does not exist in the installed version
- **Severity:** Medium (only because plans reference it)
- **Evidence:** `grep -ri photon` across all Python and Markdown in the repo: zero hits.
  BlueBubbles support exists (`gateway/platforms/bluebubbles.py`). [Verified locally]
- **Why it matters:** Your rule 13 in action: any plan that assumes Photon is in your
  installed Hermes is wrong. If Photon is an upstream feature newer than your fork point,
  getting it requires the upstream update you haven't decided on yet (F4). Whether it
  exists upstream at all could not be verified from this session. [Unverified]
- **Correction:** Defer iMessage entirely (see Section F "Do not do"). When you revisit:
  verify Photon exists in whatever version you're actually running, then evaluate its
  safety surface vs BlueBubbles (which requires a helper server with full Messages access —
  the exact contact-spam risk you've already been burned by). [BlueBubbles architecture:
  Inferred from adapter code + general knowledge; verify before use]
- **Effort:** n/a (deferral)
- **Jon approval required:** Yes before any iMessage activation.

### F6 — Model fallback chain: supported but configuration unknown
- **Severity:** Medium (becomes Critical the day Owl Alpha disappears, if unconfigured)
- **Evidence:** `agent/agent_init.py` implements an ordered `fallback_providers` chain
  (tried on connect failure/timeout/auth failure), plus per-provider
  `request_timeout_seconds` / `stale_timeout_seconds` and `api_max_retries` (default 3;
  docs recommend 1 for fast failover when fallbacks exist). [Verified locally]
  Whether your `config.yaml` configures any of it: [Unverified].
- **Why it matters:** This is the stock answer to "which no-additional-cost fallback is
  already available." If you have Nous Portal auth (`hermes login` — and your local patch
  adding the `nous-portal` alias suggests you do [Inferred]), that's a zero-new-billing
  fallback. Any other credential already in `.env` works too.
- **Correction:** On the Mac, check `config.yaml` for `fallback_providers`; if absent, add
  one entry pointing at a provider you already have credentials for, and set
  `api_max_retries: 1`.
- **Effort:** 10 min
- **Jon approval required:** Yes (config change).

### F7 — Discord access controls: mechanism verified, settings unknown
- **Severity:** Medium until verified, then likely Low
- **Evidence:** The Discord plugin (`plugins/platforms/discord/adapter.py`) reads
  `allow_from` from platform config; the gateway config layer bridges `dm_policy`,
  `allow_from`, `group_allow_from`, and a documented `guest_mode` for explicit @mentions.
  Home channel via `DISCORD_HOME_CHANNEL`. [Verified locally] Your actual values:
  [Unverified].
- **Why it matters:** If `allow_from` is empty/open, anyone who can DM the bot can drive
  Saint. Given Saint has file and (eventually) messaging powers, the allowlist is your
  primary safety boundary on the current interface.
- **Correction:** Verify on the Mac that `allow_from` contains only your Discord user ID and
  `dm_policy` is restrictive. One-line check in `config.yaml`.
- **Effort:** 5 min
- **Jon approval required:** Only if values need changing.

### F8 — Identity-file duplication risk between `~/.hermes` and the vault
- **Severity:** Medium (likely; unverifiable from here)
- **Evidence:** Your own file list shows SOUL.md, USER.md, MEMORY.md existing in *both*
  `~/.hermes`/`~/.hermes/memories` and the vault root. Hermes loads identity from its home
  directory (config-driven); a vault copy is not automatically authoritative. [Inferred
  from stock Hermes layout; your actual load paths Unverified]
- **Why it matters:** Two copies of SOUL.md is a classic competing-source-of-truth setup:
  you edit the vault copy, Saint keeps obeying the runtime copy, and the drift looks like
  "context drift" or disobedience.
- **Correction:** On the Mac, diff the pairs (`diff ~/.hermes/SOUL.md
  ~/Documents/novaaaaa/SOUL.md` etc.). Pick one authoritative location (runtime files in
  `~/.hermes`, vault holds *reference/knowledge*, not identity), and make the other a
  pointer or delete it. Recommend only — per your rule 12 I am not proposing content edits.
- **Effort:** 15 min to diff and decide
- **Jon approval required:** Yes.

### F9 — Positive: everything you need ships stock (no custom framework required)
- **Severity:** Low (informational, but it shapes every other decision)
- **Evidence:** [All Verified locally]
  - **Claude Code delegation:** stock skill v2.2.0 with two modes; print mode
    (`claude -p ... --allowedTools 'Read,Edit' --max-turns N`) is the recommended bounded,
    non-interactive handoff — exactly the "task packet" pattern you want.
  - **Multi-agent:** first-class profiles (`hermes profile create`, per-profile gateways
    with s6 supervision checks in `hermes doctor`). Franklin = a profile, not a new install.
  - **Health check:** `hermes doctor` checks version consistency, gateway/service
    supervision, provider auth, tool availability.
  - **Safety defaults:** dangerous-command approval prompts; `subagent_auto_approve`
    defaults to **deny**; shell-hook allowlist persisted to
    `~/.hermes/shell-hooks-allowlist.json`.
  - **Cron:** jobs in secured (0700) dirs, absolute `workdir` required, per-job output dirs.
  - **OpenRouter routing:** `provider_routing` supports `sort`, `only`/`ignore`, and
    `data_collection: "deny"`.
- **Why it matters:** Your instinct (rule 15) is correct and the evidence supports it:
  there is no proven need for any custom framework. The gap between your goals and your
  setup is *configuration*, not code.
- **Correction:** none — this is the baseline to defend.

### F10 — Stray planning note in repo root
- **Severity:** Low
- **Evidence:** `hermes-already-has-routines.md` sits at repo root; it arrived via an
  upstream commit (`ab706a3`), so it's upstream clutter, not yours. [Verified locally]
- **Why it matters:** Cosmetic only. Don't clean it up locally — that just adds fork drift
  (F4).
- **Correction:** Ignore it.
- **Jon approval required:** No (no action).

---

## D. Top ten improvements (ranked by value)

1. **Configure `fallback_providers` + `api_max_retries: 1`** so Owl Alpha's inevitable
   disappearance is a non-event. (F2/F6 — reduces failure risk, free)
2. **Verify the Discord `allow_from` allowlist and `dm_policy`** on the Mac. (F7 — safety)
3. **Get agent memory out of git**: gitignore `profiles/*/memories/`, `git rm --cached` the
   ledger. (F3 — prevents data exposure)
4. **Run the Mac-local verification checklist** (Section H Phase 1) to convert this report's
   Unverified items into facts. (F1)
5. **Resolve identity-file duplication** — one authoritative SOUL/USER/MEMORY location.
   (F8 — kills context drift at the root)
6. **Adopt `hermes doctor` as your standing health check** — it already exists; put it in
   muscle memory (or a cron job that only *reports*). (F9 — reliability, zero build)
7. **Use the stock claude-code skill in print mode for delegation** — task packet =
   prompt + `--allowedTools` + `--max-turns` + `workdir`; Saint validates by reading the
   diff/tests, never by trusting the transcript. (F9 — capability with no new code)
8. **Restructure local patches onto a branch** (or upstream the alias fix) so updating
   Hermes becomes routine. (F4 — maintenance)
9. **Add a host-identity guard to Saint's rules** (refuse host-specific file/system actions
   when hostname doesn't match). (F1 — wrong-host protection; needs your approval per
   rule 12)
10. **Add a sensitive-data rule for stealth/free models** — credentials, finances,
    contacts, message content never go through logging providers. (F2 — privacy, zero cost)

## E. Quick wins (<15 minutes each)

- `hermes doctor` on the Mac — instant health baseline. [command Verified locally]
- Check `config.yaml` for `allow_from`, `dm_policy`, `fallback_providers` — read-only.
- `diff ~/.hermes/SOUL.md ~/Documents/novaaaaa/SOUL.md` (and siblings) — read-only.
- Add `profiles/*/memories/` to `.gitignore` + `git rm --cached profiles/saint-mac/memories/revenue_ledger.md`.
- `hermes cron list` (or equivalent) to enumerate actual scheduled jobs.
- `claude --version && claude auth status --text` to verify Claude Code install + login.
- Set `reasoning_effort` deliberately (config default is `medium`) — cheap latency lever.

## F. Do not do

- **Do not build a custom orchestration framework.** Profiles, the claude-code skill, cron,
  and fallback chains cover Saint/Franklin/Claude delegation stock. (F9)
- **Do not activate BlueBubbles now.** It's the only iMessage path in your installed
  version, and it implies a server with full Messages access — your highest contact-spam
  surface. Photon isn't in your version at all, so there is no "safer iMessage option" to
  enable today. (F5)
- **Do not patch Hermes source further on `main`.** Every patch raises the cost of the
  upstream update you'll eventually want (e.g., for Photon). (F4)
- **Do not give Franklin a separate Discord bot or separate install** until a plain profile
  demonstrably can't do the job.
- **Do not put runtime/memory files in the source repo or the vault's identity files in two
  places.** One home per file. (F3, F8)
- **Do not let cron jobs take external actions** (messages, posts, purchases). Cron should
  schedule *reports and maintenance*; anything outward needs you in the loop.
- **Do not configure an Anthropic API key for Saint→Claude delegation** — subscription
  login via the CLI is the no-new-billing path, per your constraint.
- **Do not attempt Nova/Windows work** until the host is reachable and verified.

## G. Recommended final architecture

Simplest structure consistent with the evidence — all stock Hermes:

- **Saint (profile `saint-mac`)** — the only front door. Talks to you on Discord
  (allowlisted DMs + one home channel). Owns the vault inbox workflow. Delegates; doesn't
  build.
- **Owl Alpha** — daily driver *while it lasts*, with a configured fallback chain behind it
  (Nous Portal or any credential you already hold). Sensitive content is excluded from
  stealth models by rule.
- **Claude Code** — specialist builder, invoked by Saint via the stock skill in **print
  mode only**: one task packet in (goal, files, constraints, `--allowedTools`,
  `--max-turns`), diff + test output back. Saint reviews the diff, runs the tests, and
  reports with evidence. Interactive tmux mode reserved for you, not Saint.
- **Franklin** — a second Hermes **profile** on the same Mac install, created only when a
  concrete recurring bounded job exists. Narrow SOUL, no messaging platforms attached,
  returns evidence (diffs, logs) to Saint via files. No separate bot, no separate runtime.
- **Nova** — stays on paper until the Windows host is reachable. When activated: its own
  Hermes install on the Windows box, scoped to that machine; no Mac-side Nova runtime ever.
- **Discord** — sole live channel. Survives via launchd/supervised gateway (verify on Mac).
- **Photon/iMessage** — future-only; requires (1) updating Hermes to a version that
  actually contains it, (2) verifying its permission surface is read-scoped and
  allowlisted, (3) a dry-run period where it can receive but not send.
- **Obsidian vault** — operational knowledge and inbox only: one markdown note per idea in
  `00-Inbox/AI-Ideas/`, processed notes move to reference/archive. Identity lives in
  `~/.hermes`, not the vault.

## H. Implementation plan (do not implement without approval)

### Phase 1 — Stabilize (run on the Mac; ~45 min total)
- **Objective:** Convert Unverified → Verified; close the two highest risks (fallback,
  allowlist); stop memory-in-git.
- **Files/settings:** `~/.hermes/config.yaml`, `~/.hermes/.env` (names only),
  `~/.hermes/hermes-agent/.gitignore`, no identity files.
- **Steps (read-only first):**
  1. `hermes doctor` — capture output.
  2. `hermes --version`; `git -C ~/.hermes/hermes-agent status/log -3` — confirm it matches
     this report's fork state (`a4cf603` on v0.15.1).
  3. `launchctl list | grep -i hermes`; check gateway logs location and last error.
  4. Read `config.yaml`: model block, `fallback_providers`, `platforms.discord.allow_from`,
     `dm_policy`, home channel, cron jobs (`hermes cron list`).
  5. `grep -c '=' ~/.hermes/.env` and list variable *names* only.
  6. `claude --version && claude auth status --text`.
  7. Diff the duplicated identity files (report only).
  8. **(write, with approval)** Add fallback chain + `api_max_retries: 1`; tighten
     `allow_from` if needed; gitignore + untrack `profiles/*/memories/`.
- **Acceptance tests:** `hermes doctor` clean or explained; kill the gateway process and
  confirm launchd restarts it; send one Discord message from an allowlisted account
  (works) and confirm non-allowlisted access is rejected; temporarily set a bogus primary
  model and confirm fallback engages.
- **Rollback:** `config.yaml` is a single file — copy it aside first (`cp config.yaml
  config.yaml.bak-$(date +%F)`); revert the gitignore commit with `git revert`.
- **Approval checkpoints:** before step 8 (the only writes).

### Phase 2 — Improve (~1–2 hours, after Phase 1 green)
- **Objective:** Clean delegation and maintainable fork.
- **Files/settings:** git branches in `~/.hermes/hermes-agent`; Saint's operating notes
  (with your approval per rule 12); vault inbox folder conventions.
- **Steps:** move local patches to a `local-patches` branch (or upstream the alias fix);
  define the Claude Code task-packet convention (print mode, allowed tools, max turns,
  evidence-required review); define the inbox workflow (one note per idea; processed notes
  move; social-media claims verified against primary sources before being filed as fact);
  add the host-identity guard rule.
- **Acceptance tests:** `git pull` from upstream completes on `main` without conflicts; one
  end-to-end delegation: Saint → Claude Code print-mode task → diff returned → tests run →
  evidence posted to Discord; one inbox note processed end-to-end.
- **Rollback:** branches are non-destructive; revert instruction-file edits from the diff
  you approved.
- **Approval checkpoints:** any identity/instruction file edit; the git restructure.

### Phase 3 — Expand (only when a concrete need exists)
- **Objective:** Franklin activation; Hermes update; iMessage groundwork.
- **Files/settings:** `hermes profile create franklin`; upstream merge; Photon evaluation.
- **Steps:** create Franklin as a profile with a narrow SOUL and no platforms; route work
  to it from Saint only; update Hermes `main` from upstream once local patches are off
  `main` (Phase 2), then re-run `hermes doctor` + Phase 1 acceptance tests; only then
  re-evaluate Photon in the *new* version against the safety bar in Section G. Nova waits
  for a reachable, verified Windows host.
- **Acceptance tests:** Franklin completes one bounded task with evidence and *cannot* be
  reached from Discord; post-update doctor is clean and Discord works; Photon (if present)
  passes a receive-only dry run for a week.
- **Rollback:** profiles are deletable; pre-update git tag (`git tag pre-update-$(date
  +%F)`) allows checkout-rollback; Photon stays off by default.
- **Approval checkpoints:** Franklin creation; the upstream update; anything iMessage.

## I. Final recommendation

- **Today:** Run Phase 1 steps 1–7 (all read-only) on the Mac; approve and apply step 8
  (fallback chain, allowlist check, memory-out-of-git). ~45 minutes total.
- **This week:** Phase 2 — fork hygiene and the Claude Code delegation convention; diff and
  consolidate the duplicated identity files.
- **Postpone:** Franklin (until a concrete recurring job exists), Nova (until the Windows
  host is verified reachable), all iMessage work, any upstream Hermes update (until local
  patches are off `main`).
- **Keep Owl Alpha?** Yes, *as a free temporary primary with eyes open*: it logs your
  prompts and will disappear without notice. Keep it only behind a configured fallback and
  a no-sensitive-data rule. [Verified from OpenRouter listing]
- **Activate Franklin?** Not yet. The profile mechanism is ready whenever you are; nothing
  is gained by activating it without a queued workload.
- **Update Hermes main?** Not this week. Do Phase 2 fork hygiene first; then yes, update —
  it's also your only path to Photon if it exists upstream.
- **Is Photon safe enough yet?** Unanswerable in the best way: **it is not in your
  installed version at all** (verified, zero references in v0.15.1 source and docs), so
  there is nothing to enable safely or unsafely today. BlueBubbles exists but is the
  higher-risk architecture; leave iMessage off.

---

## Evidence index

- **Verified locally** (this repo clone): fork state `a4cf603` on Hermes v0.15.1; local
  patch contents; `profiles/saint-mac/memories/revenue_ledger.md` tracked in git;
  `.gitignore` gaps; Photon absent / BlueBubbles present; Discord plugin +
  allowlist/dm_policy/guest_mode mechanisms; `fallback_providers` chain +
  timeout/retry knobs; `hermes doctor`; claude-code skill v2.2.0; profile system;
  `subagent_auto_approve` default deny; cron security properties; OpenRouter
  `provider_routing` incl. `data_collection: deny`.
- **Verified from official documentation:** Owl Alpha pricing/context/logging/stealth
  status (openrouter.ai/openrouter/owl-alpha; OpenRouter announcement).
- **Inferred:** that you hold Nous Portal credentials (from your alias patch); BlueBubbles'
  Messages-access architecture; that identity files are duplicated runtime-vs-vault (from
  your own path list).
- **Unverified (requires Mac):** everything in `~/.hermes` runtime state, `.env` names,
  launchd, logs, scheduled cron jobs, session storage, vault contents, identity-file
  contents, Claude Code install/auth, live Discord settings, Saint's actual model setting.

*No files outside this new report were created or modified. No services were touched. No
messages were sent. No secrets were read.*
