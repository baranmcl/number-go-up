# Leaderboard Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a local-only, switchable-name leaderboard panel to the right of the existing counter, persisted in `localStorage`.

**Architecture:** All changes happen in the existing single `index.html` file. The page becomes a two-column layout (counter on the left, leaderboard panel on the right). Player identity is driven by an always-visible name input. A new `localStorage` key (`numberGoUp.leaderboard`) holds a `name → { count, lastActive }` object. A second key (`numberGoUp.activePlayer`) remembers the current player across reloads.

**Tech Stack:** HTML, inline CSS, vanilla JavaScript, `localStorage`.

**Repository:** `C:\Users\baranmcl\Code\number-go-up`, branch `main`.

**Testing note:** Manual only — same as the rest of the project. The plan ends with an end-to-end manual checklist in Task 6.

**Reference spec:** [`docs/superpowers/specs/2026-04-14-leaderboard-design.md`](../specs/2026-04-14-leaderboard-design.md)

---

## File structure

Only `index.html` and `README.md` change. The HTML file gains:

- **Markup** — a wrapper `<main class="layout">` with two children: `<section class="player-col">` (name input + counter + buttons) and `<aside class="leaderboard-col">` (leaderboard panel).
- **CSS** — flexbox two-column layout with a media query that stacks the columns under ~700px, plus styling for the name input, leaderboard card, leaderboard rows, and the highlighted active row.
- **Script** — a small data layer (`loadBoard`, `saveBoard`, `loadActivePlayer`, `saveActivePlayer`, `getEntry`, `setEntry`, `removeEntry`), a `getActiveName()` helper, a `renderLeaderboard()` function, and updated click/reset/name-input event handlers.

The previous single-player `numberGoUp.count` key is no longer used. It is intentionally left in `localStorage` (no migration). The script is rewritten to use the new keys instead.

---

### Task 1: Two-column layout, name input, and leaderboard panel scaffolding

**Goal:** Get the new visual structure on screen with no behavioral change yet — the counter still increments locally but is not yet tied to a player name. The leaderboard panel renders an empty state.

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the entire `<style>` block with the new two-column styles**

Open `index.html`. Replace the existing `<style>` block (everything between `<style>` and `</style>`) with this:

```html
    <style>
      html, body {
        margin: 0;
        height: 100%;
      }
      body {
        display: flex;
        align-items: center;
        justify-content: center;
        font-family: system-ui, -apple-system, "Segoe UI", sans-serif;
        background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
        color: #fff;
        padding: 2rem;
        box-sizing: border-box;
      }
      .layout {
        display: flex;
        align-items: center;
        gap: 3rem;
        max-width: 1100px;
        width: 100%;
      }
      .player-col {
        flex: 1 1 0;
        display: flex;
        flex-direction: column;
        align-items: center;
      }
      .leaderboard-col {
        flex: 0 0 320px;
      }
      @media (max-width: 700px) {
        body { align-items: flex-start; }
        .layout { flex-direction: column; gap: 2rem; }
        .leaderboard-col { flex: 1 1 auto; width: 100%; max-width: 360px; }
      }

      #name {
        width: 100%;
        max-width: 320px;
        font-size: 1rem;
        padding: 0.6rem 0.9rem;
        margin-bottom: 1.5rem;
        border-radius: 999px;
        border: 1px solid rgba(255, 255, 255, 0.25);
        background: rgba(255, 255, 255, 0.06);
        color: #fff;
        text-align: center;
        outline: none;
      }
      #name::placeholder { color: rgba(255, 255, 255, 0.45); }
      #name:focus { border-color: rgba(255, 255, 255, 0.55); }

      #count {
        font-size: 10rem;
        font-weight: 800;
        margin: 0 0 2rem 0;
        line-height: 1;
        transition: color 0.3s ease-out;
      }
      @keyframes pulse {
        0%   { transform: scale(1);    color: #ffffff; }
        40%  { transform: scale(1.18); color: #ffd166; }
        100% { transform: scale(1);    color: #ffffff; }
      }
      #count.pulse { animation: pulse 0.35s ease-out; }
      @keyframes shake {
        0%, 100% { transform: translateX(0); }
        20%      { transform: translateX(-10px); }
        40%      { transform: translateX(10px); }
        60%      { transform: translateX(-6px); }
        80%      { transform: translateX(6px); }
      }
      #count.shake { animation: shake 0.4s ease-in-out; }

      #click {
        font-size: 1.5rem;
        font-weight: 700;
        padding: 1rem 2.5rem;
        border: none;
        border-radius: 999px;
        background: #e94560;
        color: #fff;
        cursor: pointer;
        transition: transform 0.08s ease-out, background-color 0.15s ease-out, box-shadow 0.15s ease-out, opacity 0.15s ease-out;
        box-shadow: 0 6px 20px rgba(233, 69, 96, 0.35);
      }
      #click:hover:not(:disabled) {
        background: #ff5c77;
        box-shadow: 0 8px 26px rgba(233, 69, 96, 0.5);
      }
      #click:active:not(:disabled) {
        transform: translateY(2px) scale(0.98);
        box-shadow: 0 3px 12px rgba(233, 69, 96, 0.35);
      }
      #click:disabled, #reset:disabled {
        opacity: 0.4;
        cursor: not-allowed;
      }
      #reset {
        margin-top: 1.25rem;
        font-size: 0.9rem;
        padding: 0.5rem 1rem;
        border: 1px solid rgba(255, 255, 255, 0.25);
        border-radius: 999px;
        background: transparent;
        color: rgba(255, 255, 255, 0.7);
        cursor: pointer;
        transition: color 0.15s ease-out, border-color 0.15s ease-out, opacity 0.15s ease-out;
      }
      #reset:hover:not(:disabled) {
        color: #fff;
        border-color: rgba(255, 255, 255, 0.5);
      }

      .leaderboard {
        background: rgba(255, 255, 255, 0.06);
        border: 1px solid rgba(255, 255, 255, 0.12);
        border-radius: 1rem;
        padding: 1.25rem 1.25rem 1rem;
        backdrop-filter: blur(4px);
      }
      .leaderboard h2 {
        margin: 0 0 0.75rem 0;
        font-size: 1.1rem;
        font-weight: 700;
        letter-spacing: 0.04em;
        text-transform: uppercase;
        color: rgba(255, 255, 255, 0.85);
      }
      .leaderboard ol {
        list-style: none;
        margin: 0;
        padding: 0;
      }
      .leaderboard li {
        display: grid;
        grid-template-columns: 2rem 1fr auto;
        align-items: baseline;
        gap: 0.5rem;
        padding: 0.5rem 0.6rem;
        border-radius: 0.5rem;
        font-size: 1rem;
      }
      .leaderboard li + li { margin-top: 0.15rem; }
      .leaderboard .rank { color: rgba(255, 255, 255, 0.5); font-variant-numeric: tabular-nums; }
      .leaderboard .name { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
      .leaderboard .score { font-weight: 700; font-variant-numeric: tabular-nums; }
      .leaderboard li.active {
        background: rgba(233, 69, 96, 0.18);
        border: 1px solid rgba(233, 69, 96, 0.5);
        padding: calc(0.5rem - 1px) calc(0.6rem - 1px);
      }
      .leaderboard .empty {
        color: rgba(255, 255, 255, 0.45);
        font-style: italic;
        padding: 0.5rem 0.6rem;
      }
      .leaderboard .you-divider {
        margin: 0.5rem 0 0.25rem;
        font-size: 0.7rem;
        text-transform: uppercase;
        letter-spacing: 0.08em;
        color: rgba(255, 255, 255, 0.45);
        padding: 0 0.6rem;
      }
    </style>
```

- [ ] **Step 2: Replace the entire `<body>` content with the new markup**

Replace the existing `<body>` block in `index.html` (everything between `<body>` and `</body>`) with this:

```html
  <body>
    <main class="layout">
      <section class="player-col">
        <input
          id="name"
          type="text"
          maxlength="15"
          autocomplete="off"
          spellcheck="false"
          placeholder="Your name"
        />
        <div id="count">0</div>
        <button id="click" disabled>Click me</button>
        <button id="reset" disabled>Reset</button>
      </section>
      <aside class="leaderboard-col">
        <div class="leaderboard">
          <h2>Leaderboard</h2>
          <ol id="board"></ol>
          <div class="empty" id="empty">No clicks yet</div>
        </div>
      </aside>
    </main>
    <script>
      const nameInput = document.getElementById("name");
      const countEl = document.getElementById("count");
      const clickBtn = document.getElementById("click");
      const resetBtn = document.getElementById("reset");
      const boardEl = document.getElementById("board");
      const emptyEl = document.getElementById("empty");

      let count = 0;

      function render() {
        countEl.textContent = String(count);
      }

      function playAnimation(name) {
        countEl.classList.remove(name);
        void countEl.offsetWidth;
        countEl.classList.add(name);
      }

      clickBtn.addEventListener("click", () => {
        count += 1;
        render();
        playAnimation("pulse");
      });

      resetBtn.addEventListener("click", () => {
        count = 0;
        render();
        playAnimation("shake");
      });

      render();
    </script>
  </body>
```

Note: at this point both buttons start disabled (set in the markup). They will be wired to the name input in Task 2. The empty `<ol id="board">` plus the "No clicks yet" message together form the empty state.

- [ ] **Step 3: Manually verify**

Open `C:\Users\baranmcl\Code\number-go-up\index.html` in a browser. Confirm:
- Two-column layout: name input + counter + buttons on the left, leaderboard card with the heading "Leaderboard" and an italic muted "No clicks yet" message on the right.
- Both Click and Reset buttons are visibly disabled (faded, no hover effect).
- Resize the browser window to less than ~700px wide — the leaderboard should stack below the counter column instead of overflowing horizontally.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/baranmcl/Code/number-go-up
git add index.html
git commit -m "feat(leaderboard): scaffold two-column layout and panel"
```

---

### Task 2: Data layer, active player, and name-driven counter switching

**Goal:** Wire the name input to drive an active player. The counter loads, increments, and resets per player and persists across reloads. The leaderboard data is updated underneath, but visual rendering of the leaderboard is deferred to Task 3.

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the entire `<script>` block with the new data layer and handlers**

In `index.html`, replace the entire `<script>` block (everything between `<script>` and `</script>`, inclusive of its current contents) with this:

```html
    <script>
      const nameInput = document.getElementById("name");
      const countEl = document.getElementById("count");
      const clickBtn = document.getElementById("click");
      const resetBtn = document.getElementById("reset");
      const boardEl = document.getElementById("board");
      const emptyEl = document.getElementById("empty");

      const BOARD_KEY = "numberGoUp.leaderboard";
      const ACTIVE_KEY = "numberGoUp.activePlayer";

      function loadBoard() {
        try {
          const raw = localStorage.getItem(BOARD_KEY);
          if (!raw) return {};
          const parsed = JSON.parse(raw);
          return (parsed && typeof parsed === "object") ? parsed : {};
        } catch {
          return {};
        }
      }

      function saveBoard(board) {
        localStorage.setItem(BOARD_KEY, JSON.stringify(board));
      }

      function loadActivePlayer() {
        return localStorage.getItem(ACTIVE_KEY) || "";
      }

      function saveActivePlayer(name) {
        if (name) {
          localStorage.setItem(ACTIVE_KEY, name);
        } else {
          localStorage.removeItem(ACTIVE_KEY);
        }
      }

      let board = loadBoard();

      function getActiveName() {
        return nameInput.value.trim();
      }

      function getEntry(name) {
        return board[name] || null;
      }

      function setEntry(name, count) {
        board[name] = { count, lastActive: Date.now() };
        saveBoard(board);
      }

      function removeEntry(name) {
        delete board[name];
        saveBoard(board);
      }

      function currentCount() {
        const name = getActiveName();
        if (!name) return 0;
        const entry = getEntry(name);
        return entry ? entry.count : 0;
      }

      function renderCount() {
        countEl.textContent = String(currentCount());
      }

      function renderLeaderboard() {
        // Implemented in Task 3.
      }

      function updateButtonsEnabled() {
        const hasName = getActiveName().length > 0;
        clickBtn.disabled = !hasName;
        resetBtn.disabled = !hasName;
      }

      function playAnimation(name) {
        countEl.classList.remove(name);
        void countEl.offsetWidth;
        countEl.classList.add(name);
      }

      nameInput.addEventListener("input", () => {
        saveActivePlayer(getActiveName());
        updateButtonsEnabled();
        renderCount();
        renderLeaderboard();
      });

      clickBtn.addEventListener("click", () => {
        const name = getActiveName();
        if (!name) return;
        const entry = getEntry(name);
        const next = (entry ? entry.count : 0) + 1;
        setEntry(name, next);
        renderCount();
        renderLeaderboard();
        playAnimation("pulse");
      });

      resetBtn.addEventListener("click", () => {
        const name = getActiveName();
        if (!name) return;
        removeEntry(name);
        renderCount();
        renderLeaderboard();
        playAnimation("shake");
      });

      // Initial state on page load.
      nameInput.value = loadActivePlayer();
      updateButtonsEnabled();
      renderCount();
      renderLeaderboard();
    </script>
```

- [ ] **Step 2: Manually verify**

Refresh `index.html` in the browser. Confirm:
- On a fresh load (or after clearing site data), the name input is empty and both buttons are disabled.
- Type "Alice" into the name input — both buttons become enabled, counter shows `0`.
- Click "Click me" several times — counter increments and pulses on each click.
- Clear the name input — counter shows `0`, both buttons go back to disabled.
- Type "Alice" again — counter shows the count Alice had reached.
- Type "Bob" instead — counter shows `0` (Bob is new). Click a few times. Then re-type "Alice" — Alice's count reappears.
- Click Reset while "Alice" is active — counter shows `0`, shake animation plays.
- Switch back to "Bob" — Bob's earlier count is still there.
- Refresh the page — the name input is pre-filled with whichever name was active, and the counter shows that player's count.

(The leaderboard panel still shows "No clicks yet" — that gets wired up in Task 3.)

- [ ] **Step 3: Commit**

```bash
cd /c/Users/baranmcl/Code/number-go-up
git add index.html
git commit -m "feat(leaderboard): per-player counter driven by name input"
```

---

### Task 3: Render the leaderboard

**Goal:** Replace the placeholder `renderLeaderboard()` function with a real implementation that draws the top 10 entries plus a "you" row when the active player is below rank 10. Highlight the active player's row.

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the placeholder `renderLeaderboard` function**

In `index.html`, find this block in the `<script>`:

```js
      function renderLeaderboard() {
        // Implemented in Task 3.
      }
```

Replace it with:

```js
      function sortedEntries() {
        return Object.entries(board)
          .map(([name, entry]) => ({ name, count: entry.count, lastActive: entry.lastActive || 0 }))
          .sort((a, b) => {
            if (b.count !== a.count) return b.count - a.count;
            return b.lastActive - a.lastActive;
          });
      }

      function rowHtml(rank, entry, isActive) {
        const cls = isActive ? "active" : "";
        const safeName = entry.name
          .replace(/&/g, "&amp;")
          .replace(/</g, "&lt;")
          .replace(/>/g, "&gt;");
        return `<li class="${cls}"><span class="rank">${rank}</span><span class="name">${safeName}</span><span class="score">${entry.count}</span></li>`;
      }

      function renderLeaderboard() {
        const entries = sortedEntries();
        const activeName = getActiveName();

        if (entries.length === 0) {
          boardEl.innerHTML = "";
          emptyEl.style.display = "";
          return;
        }
        emptyEl.style.display = "none";

        const top = entries.slice(0, 10);
        const activeInTop = activeName && top.some((e) => e.name === activeName);
        const activeEntry = activeName ? entries.find((e) => e.name === activeName) : null;
        const activeRank = activeEntry ? entries.findIndex((e) => e.name === activeName) + 1 : null;

        let html = top
          .map((entry, i) => rowHtml(i + 1, entry, entry.name === activeName))
          .join("");

        if (activeEntry && !activeInTop) {
          html += `<li class="you-divider">You</li>`;
          html += rowHtml(activeRank, activeEntry, true);
        }

        boardEl.innerHTML = html;
      }
```

Note: the `<li class="you-divider">` reuses the `<li>` element type so it sits naturally inside the `<ol>`, but its CSS strips list styling and gives it a small uppercase label. The CSS for `.you-divider` was already added in Task 1.

- [ ] **Step 2: Manually verify**

Refresh `index.html`. Confirm:
- With an empty board, the panel still shows "No clicks yet".
- Type "Alice" and click 5 times — a single row appears in the leaderboard: `1  Alice  5`, highlighted (red-tinted background and border).
- Type "Bob" and click 8 times — leaderboard now shows `1 Bob 8` (highlighted, since Bob is active) and `2 Alice 5` (not highlighted).
- Switch back to Alice (re-type her name) — same two rows, but now Alice's row is highlighted instead of Bob's.
- Click Reset while Alice is active — Alice's row vanishes from the board, and the counter shows `0`. Bob's row remains as `1 Bob 8`.
- Add 11 distinct names with varying counts (e.g., A=1, B=2, … K=11). Set the active name to the lowest scorer ("A" with count 1) — confirm: top 10 rows are visible at the top, then a "YOU" divider, then a single row showing A's actual rank (11) and count (1), highlighted.
- Switch the active name to "K" (count 11) — confirm: K appears highlighted at rank 1 within the top 10, and there is no "YOU" divider section.

- [ ] **Step 3: Commit**

```bash
cd /c/Users/baranmcl/Code/number-go-up
git add index.html
git commit -m "feat(leaderboard): render entries with active highlight and you-row"
```

---

### Task 4: Update README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Replace the contents of `README.md`**

```markdown
# number-go-up

A tiny single-page web app: type a name, click a button, watch the number go up. Built as a vibe-coding exercise.

Each name is its own player with its own counter, all persisted in your browser's `localStorage`. A leaderboard on the right shows the top 10 players and always shows your row, even if you're not in the top 10. Hit Reset to remove your entry from the leaderboard and zero out your counter — other players are untouched.

The leaderboard is local to your browser only. If a friend visits the page on their own device, they get their own empty leaderboard.

## Running it

Open `index.html` in any modern web browser. That's it — no build step, no dependencies.

A hosted copy lives at https://baranmcl.github.io/number-go-up/.
```

- [ ] **Step 2: Commit**

```bash
cd /c/Users/baranmcl/Code/number-go-up
git add README.md
git commit -m "docs: describe leaderboard in README"
```

---

### Task 5: Push to GitHub Pages

**Goal:** Deploy the new feature so the hosted site at `https://baranmcl.github.io/number-go-up/` is updated.

- [ ] **Step 1: Push**

```bash
cd /c/Users/baranmcl/Code/number-go-up
git push origin main
```

- [ ] **Step 2: Wait ~30–60 seconds for GitHub Pages to rebuild, then open the hosted URL**

Open `https://baranmcl.github.io/number-go-up/` in a browser. Confirm the new two-column layout with the leaderboard panel is visible.

(No commit — pushing is the deployment.)

---

### Task 6: Final end-to-end manual check

**Goal:** Walk the full feature once locally to make sure nothing regressed.

- [ ] **Step 1: Open `index.html` fresh and run through this checklist**

1. Both buttons start disabled with the name input empty.
2. Type "Alice", confirm buttons enable, counter shows 0 (or her saved value if returning).
3. Click 3 times — counter increments and pulses, leaderboard shows `1 Alice 3` highlighted.
4. Switch to "Bob", click 5 times — leaderboard shows `1 Bob 5` highlighted, `2 Alice 3` not highlighted.
5. Refresh the page — the name input still shows "Bob", counter shows 5, leaderboard intact.
6. Clear the name input — counter shows 0, both buttons disable. The leaderboard still shows the two existing rows, neither highlighted.
7. Re-type "Alice" — Alice's row becomes highlighted, counter shows 3.
8. Click Reset — Alice's row disappears, counter shows 0, shake animation plays, only Bob's row remains.
9. Resize browser to under ~700px wide — leaderboard stacks below the counter column.

- [ ] **Step 2: No commit needed** — verification only.

---

## Done

At this point the leaderboard feature is fully implemented, deployed, and verified end-to-end. The app is still a single `index.html` with no dependencies and no build step.
