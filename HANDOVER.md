# The Compliance Climb — Handover Doc

**Purpose:** a Snakes & Ladders browser game, single HTML file, no backend. Built for Keka
(payroll/compliance HR software) as a playful marketing prototype: snakes = manual compliance
mishaps, ladders = Keka automating something.

**Current mode: vs-Competitor race.** This is NOT the original solo-climb game anymore — it was
fully replaced. You (a dog token) race a computer-controlled competitor (a cat token) to square
100. Same rules apply to both sides. First to land exactly on 100 wins; the other side gets a
loss screen. See "What's actually live right now" below for full mechanics.

## Read this first: the one thing that WILL be wrong in a fresh session

Working copies were edited inside Claude's **scratchpad temp directory**, unique to each
conversation, gone once that conversation ends. **Any path under
`...\AppData\Local\Temp\claude\...\scratchpad\...` from a previous session no longer exists.**

**The deploy repo is the only thing that survives.** In a fresh session:
1. Read `index.html` from the deploy repo path (or `git show HEAD:index.html`) — that's the real
   current live game, full stop.
2. To preview a change as a Claude Artifact, copy the file into whatever fresh scratchpad path
   *this* session provides, then Artifact-publish that copy.
3. Never assume a scratchpad path, or a standalone mockup file mentioned below, still exists —
   rebuild it from scratch if needed.

## Paths and URLs

| Copy | Path/URL | Notes |
|---|---|---|
| **Deploy repo** (source of truth) | `D:\Work Folder\ContentLabHQ\Automation Project\compliance-climb-deploy\index.html` | Git-tracked, pushed to GitHub Pages. Check this path still resolves — the whole project folder has moved drives once already this project (`C:\...` → `D:\...`). |
| **Production (live)** | https://dibyajyotidasgupta.github.io/compliance-and-ladder/ | What the public sees. Currently serving the vs-competitor game. |
| **Main Claude Artifact** | https://claude.ai/code/artifact/96619a4a-26bf-4f71-bc64-c28c52be025f | Mirrors production. Republish to this same URL when previewing changes (pass `url` explicitly if publishing from a different session/conversation than the one that made it). |
| **Mobile-fix mockup Artifact** (still open decision, see below) | https://claude.ai/code/artifact/348decd0-990c-4bc2-9b64-d0de020c670a | "Mobile Layout Options" — standalone comparison of 3 candidate fixes for a live mobile bug. NOT part of production. The source file (`mobile-fix-options.html`) lived only in scratchpad and will need to be rebuilt from this doc's description if a fresh session needs to touch it further. |

## Standard workflow for making a change (battle-tested, repeat this)

1. Edit `index.html` directly in the deploy repo.
2. To preview as a Claude Artifact before shipping: copy the file into a scratchpad path, then
   use the Artifact tool (title "The Compliance Climb", favicon 🎲). Republish to the URL above.
3. Test locally before committing:
   - Copy `index.html` to a sibling file one level up (e.g. `compliance-climb.html`) so local
     testing doesn't collide with the name GitHub Pages expects.
   - Create `.claude/launch.json` **inside `compliance-climb-deploy` itself** (the preview tool
     resolves relative to wherever Bash's cwd happens to be, which has consistently been this
     folder):
     ```json
     {
       "version": "0.0.1",
       "configurations": [{
         "name": "compliance-climb",
         "runtimeExecutable": "npx",
         "runtimeArgs": ["-y", "serve", "..", "-l", "5177"],
         "port": 5177
       }]
     }
     ```
   - `preview_start` with that config name, navigate to `http://localhost:5177/<test-file>.html`
     (use `force: true` — plain navigate has been flaky).
   - Clean up the test file and `.claude` folder afterward.
4. Commit and push:
   ```bash
   git add index.html
   git commit -m "..."
   git push origin main
   ```
   Git credentials are cached (Windows Credential Manager) — push just works, no auth prompt.
5. Confirm production actually updated (GitHub Pages takes ~1-2 min):
   ```bash
   timeout 90 bash -c 'until curl -s "https://dibyajyotidasgupta.github.io/compliance-and-ladder/?v=$(date +%s)" | grep -q "<unique string from your change>"; do sleep 5; done; echo FOUND'
   ```
6. **Never push to production without the user explicitly confirming** after a mock/artifact
   review. No exceptions, this has held all project long — the one time it was overridden was an
   explicit "You can push this to production" for the vs-competitor mode.

## A real environment limitation — plan around it

The browser preview tool's **screenshot/zoom capture has not worked all project**
("Browser pane is not displayed, so the page is not compositing frames"). An isolated test also
showed CSS animations don't progress under `getComputedStyle` polling either — the tab isn't
compositing frames at all, not just failing to screenshot. **Workaround: verify everything
structurally instead of visually:**
- `getBoundingClientRect()` / `getBBox()` for layout and geometry
- Real DOM presence/absence over real elapsed time (`performance.now()`) for timing (toasts,
  animation duration)
- Console error checks
- Actual game-state assertions (square number, turn count) after simulated/scripted play

Test early in a fresh session whether this is still broken before relying on the workaround.

## GitHub repo details

- Repo: `https://github.com/DibyajyotiDasgupta/compliance-and-ladder` (public, deliberately —
  theft risk was discussed and accepted; don't change visibility without asking)
- Branch: `main` (only branch)
- GitHub Pages: enabled, deploys from `main` / `/ (root)`
- Local git identity (set locally, not global): `Dibyajyoti Dasgupta` / `dj.deb23@gmail.com`
- `.gitignore` and `LICENSE` (proprietary, all-rights-reserved — NOT open source) already
  committed, don't remove or relicense.
- "LF will be replaced by CRLF" warnings on git commands are cosmetic, not errors.

## What's actually live right now (vs-competitor mode)

- **Two tokens race simultaneously**: you (dog, Shih Tzu design) vs. a computer competitor (cat).
  Identical rules for both sides — same snakes/ladders, same win condition.
- **Board**: 10×10 boustrophedon numbering, illustrated snakes (rotating warm-color palette,
  faces, tongues) and purple ladders, trophy at square 100. One ladder endpoint was relocated
  (43→75) earlier to reduce visual crowding.
- **Win condition**: exact roll required to land on 100 — overshoot doesn't move the token, turn
  still counts ("So close!" toast). First side to hit exactly 100 wins; shows a win screen for
  the winner, loss screen for the other side. "Rematch" button resets both.
- **Six-roll bonus/cancellation rule** (classic Snakes & Ladders rule, confirmed against a rule
  the user quoted from Google): rolling a 6 grants another roll. Three consecutive 6s cancels all
  three and sends the token back to wherever it was **before the first of the three sixes**
  (checkpoint-based, not back to square 1) — shown via an "Oops, three sixes — back to..." toast.
  For the human player, each bonus roll requires an explicit re-click of the roll button (feels
  more in-control); for the computer, bonus rolls auto-chain via an internal loop
  (`runComputerTurn()`). Verified via deterministic `Math.random` overrides (forced values at
  specific call-count positions) plus a 100k-game Monte Carlo simulation — three-consecutive-sixes
  shows up in ~13% of full races, confirmed "exceedingly rare" per attempt but not rare
  per-full-game, matching what the user wanted checked.
- **Desktop-only spacebar-to-roll**: `keydown` listener on `e.code==='Space'`, gated to
  `window.innerWidth > 600`, with `preventDefault()`.
- **Toasts**: actor-named (distinguishes "You" vs "Competitor" events), anchored above the landing
  square on desktop, fixed bottom-center on mobile.
- **End screen**: 3 stats — Snake bites, Keka rescues, Turns taken.
- **Credit line**: "Concept & build by Dibyajyoti Dasgupta" (don't remove).

## Open item #1 (active, in-progress): mobile bug — status panel blocks the board

**The bug (confirmed live via user's phone screenshot):** on mobile, the "You / Competitor"
status panel completely blocks the board because `.panel-card`'s base
`display:flex; flex-direction:column;` was never overridden back to a compact row layout for the
new two-player bar (unlike the old solo game's from-scratch compact mobile rule). **This has NOT
been fixed in production yet** — only diagnosed and mocked up.

**Process so far:** user asked to brainstorm fixes before touching code, then asked for a real
side-by-side mockup of three options (built as standalone `mobile-fix-options.html`, published as
the "Mobile Layout Options" artifact above, with an A/B/C switcher bar and a dynamic mode-label
banner so the active option is unambiguous on a real phone with zero scrolling).

**The three options:**
- **A — "Normal flow"**: status cards placed above the board as normal block content (not fixed).
  **Rejected by user** — no reason needed beyond "completely out."
- **B — "Compact strip"**: status strip joins the die + roll button in the existing fixed bottom
  bar (row layout). **User likes this**: "minimalistic and pretty clean."
- **C — "Board overlay"**: small floating badge pills in the board's top corners (`position:
  absolute` inside `.board-card`). **User likes the concept** — "spells out who's who" — **but is
  concerned the floating badges obstruct the board's number labels underneath them** (see the
  phone screenshot the user sent: the "You: 1" and "Competitor: 1" badges visually sit on top of
  the top-row square numbers 91–100).

**Decision was interrupted mid-discussion** (user was cut off / hit Stop before finishing their
comparison) — **no final pick has been made.** Next session should either:
1. Ask the user to just finish picking between B and C (A is confirmed dead), or
2. If C is still appealing, propose a concrete fix for the overlap first (e.g. shrink the badges
   further, move them to float just *above* the board's top edge instead of on top of the first
   row of squares, reduce opacity, or use a smaller pill with just an icon + number rather than
   full "You: N" text) and show that as a refined C before asking again.

Once a direction is confirmed: implement it in actual production (`index.html`), following the
standard workflow above (test locally, artifact preview optional, commit, push, confirm live).
The mobile-fix-options.html mockup file itself is throwaway scratch — don't try to preserve or
port it wholesale, just carry the CSS approach it proved out into `index.html`.

## Open item #2 (deferred, approved in principle, NOT applied)

- **Rename title**: "The Compliance Climb" → "The Compliance Game". User explicitly said
  **"do not apply yet"** — needs an explicit go-ahead before editing.
- **Reduce hero heading/subtext font sizes** — same status, approved in principle, not applied,
  waiting on explicit go-ahead.
- Do both together when the user confirms, since they were raised in the same breath.

## Open item #3 (deferred, no scope decided)

**Music** — user referenced https://snake-skip.lovable.app/ as a rough example of the vibe
wanted, then said **"We'll handle the music later."** No decision on SFX vs. background loop vs.
both, no asset sourcing discussed yet. Don't start on this unless the user brings it back up.

## Things NOT to do without checking with the user

- Don't change repo visibility (stays public, deliberately).
- Don't add an open-source license — LICENSE is deliberately proprietary.
- Don't push to production without explicit confirmation after a mock/artifact review — no
  exceptions found all project except the one explicit "push this to production" for the
  vs-competitor mode.
- Don't apply the title/font-size change (open item #2) without an explicit go-ahead, even though
  it was "approved in principle."
- Don't start on music (open item #3) unprompted.
