# The Compliance Climb — Handover Doc

**Purpose:** a Snakes & Ladders browser game, single HTML file, no backend. Built for Keka
(payroll/compliance HR software) as a playful marketing prototype: snakes = manual compliance
mishaps, ladders = Keka automating something. Player rolls dice, climbs to square 100.

## Read this first: the one thing that WILL be wrong in a fresh session

During development, edits were made to a working copy inside Claude's **scratchpad temp
directory**, which is unique to each conversation and does **not carry over** to a new session.
That path no longer exists once this conversation ends. **Do not try to reuse it.**

**The actual source of truth that survives is the deploy repo below.** In a fresh session:
1. Read `index.html` from the deploy repo path (or `git show HEAD:index.html`) to get the current
   live game.
2. If you need to publish it as a Claude Artifact again, copy that file into whatever fresh
   scratchpad path the new session provides, then use the Artifact tool on that copy.
3. Never assume any path from a previous session's scratchpad still exists.

## The three "copies" of this game and how they relate

| Copy | Path | Purpose | Persists across sessions? |
|---|---|---|---|
| **Deploy repo** (source of truth) | `D:\Work Folder\ContentLabHQ\Automation Project\compliance-climb-deploy\index.html` | Git-tracked, pushed to GitHub Pages | Yes — real disk path, git history |
| **Production (live)** | https://dibyajyotidasgupta.github.io/compliance-and-ladder/ | What the public sees | Yes — hosted on GitHub |
| **Claude Artifact (mockup/preview)** | https://claude.ai/code/artifact/96619a4a-26bf-4f71-bc64-c28c52be025f | Used to preview changes before pushing to production | Yes, the URL persists (tied to the account, not the session) — but re-publishing it requires a *local file* to point the Artifact tool at, and that file lived in the old scratchpad path |

**Important path note:** this whole project folder moved once already this session, from
`C:\Work Folder\...` to `D:\Work Folder\...` (same relative path, different drive). If paths
below don't resolve, run `pwd` / check `D:\Work Folder\ContentLabHQ\Automation Project\` first —
it may have moved again.

## Standard workflow for making a change (established and battle-tested this session)

1. Edit `index.html` directly in the deploy repo
   (`D:\Work Folder\ContentLabHQ\Automation Project\compliance-climb-deploy\index.html`).
2. If you want to preview it as a Claude Artifact before shipping: copy that file into a
   scratchpad path, then use the Artifact tool (title "The Compliance Climb", favicon 🎲,
   description: "A branded Snakes & Ladders game where manual payroll chaos gets automated
   away by Keka."). **Always republish to the same artifact URL above** (pass no `url` param if
   publishing from the same conversation that owns it; if a different session, pass the `url`
   explicitly to update in place rather than creating a duplicate).
3. Test locally before committing:
   - Copy `index.html` to a sibling file (e.g. `compliance-climb.html`) one level up, since
     GitHub Pages/production serves it as `index.html` but local testing is easier from a
     differently-named file to avoid confusion.
   - Create `.claude/launch.json` **inside the `compliance-climb-deploy` folder itself**
     (quirk: the preview tool resolves relative to wherever the Bash tool's cwd happens to be,
     which has consistently been this folder) with:
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
     (serves the parent folder so both the deploy repo and any sibling test file are reachable)
   - `preview_start` with that config name, then navigate to
     `http://localhost:5177/<test-file>.html` (use `force: true` on navigate — plain navigate
     calls were flaky this session for no clear reason).
   - Clean up afterward: delete the test file and the `.claude` folder.
4. Commit and push:
   ```bash
   git add index.html
   git commit -m "..."
   git push origin main
   ```
   Git credentials are cached (Windows Credential Manager, `credential.helper=manager`) — no
   browser auth prompt needed, push just works.
5. Confirm production actually updated (GitHub Pages takes ~1-2 min to rebuild):
   ```bash
   timeout 90 bash -c 'until curl -s "https://dibyajyotidasgupta.github.io/compliance-and-ladder/?v=$(date +%s)" | grep -q "<some string unique to your change>"; do sleep 5; done; echo FOUND'
   ```

## A real limitation hit repeatedly this session — plan around it

The browser preview tool's **screenshot capture did not work all session** ("Browser pane is not
displayed, so the page is not compositing frames"). A follow-up isolated test also showed that
**CSS animations don't progress in `getComputedStyle` reads either** — the tab wasn't actually
compositing frames at all, not just failing to screenshot. Workaround used throughout: verify
everything structurally instead —
- `getBoundingClientRect()` / `getBBox()` for layout and geometry
- Real DOM presence/absence over real elapsed time (via `performance.now()`) for anything
  timing-related (toast appear/disappear, animation duration)
- Console error checks
- Actual game-state assertions (square number, turn count, etc.) after simulated play

This may or may not still be broken in a fresh session — worth testing early (try a screenshot;
if it fails with that same message, fall back to the structural-verification approach rather
than repeatedly retrying screenshots).

## GitHub repo details

- Repo: `https://github.com/DibyajyotiDasgupta/compliance-and-ladder` (public)
- Owner: DibyajyotiDasgupta
- Branch: `main` (only branch)
- GitHub Pages: enabled, deploys from `main` / `/ (root)`
- Local git identity for this repo (set locally, not global):
  name `Dibyajyoti Dasgupta`, email `dj.deb23@gmail.com`
- `.gitignore` and `LICENSE` (proprietary/all-rights-reserved, explicitly *not* open source) are
  already committed — don't remove them.
- Line-ending warnings ("LF will be replaced by CRLF") on every git command are cosmetic/harmless,
  not an error.

## Current state as of this handover (all pushed, confirmed live)

Commit history, newest first:
1. Anchor toast to the landing square on desktop, remove board captions, enlarge toast
2. Replace token with a Shih Tzu design
3. Rename stat labels to Snake bites / Keka rescues
4. Require exact roll to reach square 100 (overshoot doesn't move the token)
5. Add turns-taken as a third end-screen metric
6. Compact mobile control bar; relocate two crowded ladder endpoints
7. Fix missing viewport meta tag; simplify board captions and pin roll button on mobile
8. Add creator credit line ("Concept & build by Dibyajyoti Dasgupta")
9. Add .gitignore and proprietary license notice
10. Initial commit

Notable design decisions baked into the current build:
- Board: 10×10 boustrophedon numbering, illustrated snakes (rotating warm-color palette, faces,
  tongues) and purple ladders, trophy icon at square 100.
- No on-board caption text anymore (removed for clutter) — story text only appears via: (a) hover
  tooltip (native SVG `<title>`, desktop only, mouse-dependent) and (b) a toast on landing.
- Toast: bigger than original (1.1rem font, 20px/26px padding), bouncy entrance, ~3.9s hold. On
  desktop it's anchored to appear right above wherever the token just landed (real math, not
  fixed position); on mobile it's untouched — fixed bottom-center, smaller.
- Mobile (≤600px breakpoint): dice + roll button merged into one compact bar pinned to the
  bottom of the screen; stats grid and "Your turn" label hidden to save space.
- Win condition: exact roll required to land on 100 (overshoot = no move, turn still counts,
  "So close!" toast).
- End screen shows 3 stats: Snake bites, Keka rescues, Turns taken.

## Open items — not yet decided, mid-conversation when handover was requested

1. **Hero title/tagline copy** — currently "The Compliance Climb" / "Roll the dice. Dodge the
   chaos of manual payroll compliance. Climb toward square 100 — where every rule is already
   handled." User asked for better name + copy; I proposed keeping the title (has good
   alliteration) and swapping the tagline to one that explains the snake/ladder mechanic
   directly: "Every snake's a compliance slip-up. Every ladder's Keka catching it. Roll and find
   out which one you hit." **No decision made yet.**
2. **Anchored-toast sizing feedback** — the desktop anchored toast was sized narrower (480px)
   than the earlier "bigger" fixed version (560px) since it's now a contextual callout rather
   than a banner. User was asked to check whether it should be wider on the live artifact — no
   confirmation received yet.

## Things NOT to do without checking with the user

- Don't rename/change the repo visibility (public, deliberately — user decided theft risk was
  overblown, confirmed real intent is "public but not likely to be stumbled on").
- Don't add an open-source license (MIT etc.) — the LICENSE is deliberately proprietary.
- Don't touch mobile view without explicit ask — user said "it works in its current state,"
  multiple times.
- Don't push to production without the user explicitly confirming after a mock/artifact review —
  this has been the pattern every single time all session, no exceptions.
