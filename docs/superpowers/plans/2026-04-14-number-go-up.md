# number-go-up Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file web page with a playful animated click counter that persists in localStorage and supports reset.

**Architecture:** One `index.html` file containing all markup, inline CSS, and inline JavaScript. No build system, no dependencies. Opened directly in a browser. Count is stored in `localStorage` under the key `numberGoUp.count`.

**Tech Stack:** HTML, CSS (inline), vanilla JavaScript (inline), `localStorage`.

**Testing note:** The spec explicitly calls for manual testing only — no test framework. Each task ends with a manual verification step (open `index.html` in a browser and confirm the described behavior).

**Repository:** `C:\Users\baranmcl\Code\number-go-up` (branch: `main`).

---

### Task 1: Create the minimal HTML skeleton with count and button

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create `index.html` with a bare-bones layout**

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>number-go-up</title>
    <style>
      html, body {
        margin: 0;
        height: 100%;
      }
      body {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        font-family: system-ui, -apple-system, "Segoe UI", sans-serif;
        background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
        color: #fff;
      }
      #count {
        font-size: 10rem;
        font-weight: 800;
        margin: 0 0 2rem 0;
        line-height: 1;
      }
      #click {
        font-size: 1.5rem;
        font-weight: 700;
        padding: 1rem 2.5rem;
        border: none;
        border-radius: 999px;
        background: #e94560;
        color: #fff;
        cursor: pointer;
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
      }
    </style>
  </head>
  <body>
    <div id="count">0</div>
    <button id="click">Click me</button>
    <button id="reset">Reset</button>
    <script>
      const countEl = document.getElementById("count");
      const clickBtn = document.getElementById("click");
      const resetBtn = document.getElementById("reset");

      let count = 0;

      function render() {
        countEl.textContent = String(count);
      }

      clickBtn.addEventListener("click", () => {
        count += 1;
        render();
      });

      render();
    </script>
  </body>
</html>
```

- [ ] **Step 2: Manually verify**

Open `C:\Users\baranmcl\Code\number-go-up\index.html` in a browser. Confirm:
- Dark gradient background, number `0` centered near the middle, red "Click me" button below it, a subdued "Reset" button under that.
- Clicking "Click me" increments the number (1, 2, 3, …).
- Reset button does nothing yet (expected — added in Task 4).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add basic click counter page"
```

---

### Task 2: Add hover and press effects to the primary button

**Files:**
- Modify: `index.html` (add CSS rules inside the existing `<style>` block)

- [ ] **Step 1: Add hover/active styles for `#click`**

Inside the `<style>` block in `index.html`, add these rules immediately after the existing `#click { … }` block:

```css
#click {
  transition: transform 0.08s ease-out, background-color 0.15s ease-out, box-shadow 0.15s ease-out;
  box-shadow: 0 6px 20px rgba(233, 69, 96, 0.35);
}
#click:hover {
  background: #ff5c77;
  box-shadow: 0 8px 26px rgba(233, 69, 96, 0.5);
}
#click:active {
  transform: translateY(2px) scale(0.98);
  box-shadow: 0 3px 12px rgba(233, 69, 96, 0.35);
}
#reset:hover {
  color: #fff;
  border-color: rgba(255, 255, 255, 0.5);
}
```

Note: the new `#click { … }` block with `transition` and `box-shadow` merges with the earlier rule — both will apply. Leave the earlier block alone.

- [ ] **Step 2: Manually verify**

Refresh `index.html` in the browser. Confirm:
- Hovering the primary button brightens it and increases the glow.
- Pressing it makes it visibly depress.
- Hovering the reset button makes its text and border brighter.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add hover and press effects to buttons"
```

---

### Task 3: Add the pulse animation on increment

**Files:**
- Modify: `index.html` (add CSS keyframes + a class, and trigger it from the click handler)

- [ ] **Step 1: Add pulse keyframes and `.pulse` class to the `<style>` block**

Add these rules to the `<style>` block, after the existing `#count { … }` rule:

```css
#count {
  transition: color 0.3s ease-out;
}
@keyframes pulse {
  0%   { transform: scale(1);    color: #ffffff; }
  40%  { transform: scale(1.18); color: #ffd166; }
  100% { transform: scale(1);    color: #ffffff; }
}
#count.pulse {
  animation: pulse 0.35s ease-out;
}
```

- [ ] **Step 2: Trigger the pulse class on each increment**

In the `<script>` block, replace the existing click handler:

```js
clickBtn.addEventListener("click", () => {
  count += 1;
  render();
});
```

with:

```js
function playAnimation(name) {
  countEl.classList.remove(name);
  // Force reflow so re-adding the class restarts the animation.
  void countEl.offsetWidth;
  countEl.classList.add(name);
}

clickBtn.addEventListener("click", () => {
  count += 1;
  render();
  playAnimation("pulse");
});
```

- [ ] **Step 3: Manually verify**

Refresh `index.html`. Click the button several times rapidly. Confirm:
- Each click makes the number briefly scale up and flash yellow-gold, then return to white.
- Rapid clicks restart the animation cleanly (no stuck-in-the-middle states).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: pulse the counter on each increment"
```

---

### Task 4: Wire up the Reset button with a shake animation

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add shake keyframes and `.shake` class**

Add these rules to the `<style>` block, after the `#count.pulse` rule:

```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%      { transform: translateX(-10px); }
  40%      { transform: translateX(10px); }
  60%      { transform: translateX(-6px); }
  80%      { transform: translateX(6px); }
}
#count.shake {
  animation: shake 0.4s ease-in-out;
}
```

- [ ] **Step 2: Add a reset click handler**

In the `<script>` block, add this after the existing `clickBtn.addEventListener(...)` block:

```js
resetBtn.addEventListener("click", () => {
  count = 0;
  render();
  playAnimation("shake");
});
```

- [ ] **Step 3: Manually verify**

Refresh `index.html`. Click the primary button a few times, then click Reset. Confirm:
- The number returns to `0`.
- The number briefly shakes left–right.
- Clicking Reset when the count is already `0` still shakes (that's fine).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: reset button zeroes count with shake animation"
```

---

### Task 5: Persist the count in localStorage

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Load the stored count on startup and save on every change**

In the `<script>` block, replace:

```js
let count = 0;

function render() {
  countEl.textContent = String(count);
}
```

with:

```js
const STORAGE_KEY = "numberGoUp.count";

function loadCount() {
  const raw = localStorage.getItem(STORAGE_KEY);
  const parsed = Number.parseInt(raw, 10);
  return Number.isFinite(parsed) ? parsed : 0;
}

function saveCount() {
  localStorage.setItem(STORAGE_KEY, String(count));
}

let count = loadCount();

function render() {
  countEl.textContent = String(count);
}
```

Then update both event handlers to call `saveCount()` after mutating `count`. The click handler becomes:

```js
clickBtn.addEventListener("click", () => {
  count += 1;
  saveCount();
  render();
  playAnimation("pulse");
});
```

And the reset handler becomes:

```js
resetBtn.addEventListener("click", () => {
  count = 0;
  saveCount();
  render();
  playAnimation("shake");
});
```

- [ ] **Step 2: Manually verify**

Open `index.html`. Confirm:
- Click the button several times (e.g., to 7).
- Refresh the page — the number should still read 7.
- Click more (e.g., to 12), refresh again — should read 12.
- Click Reset, refresh — should read 0.
- Open the browser devtools → Application → Local Storage and confirm the key `numberGoUp.count` holds the current value as a string.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: persist counter in localStorage"
```

---

### Task 6: Add a README

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write `README.md`**

```markdown
# number-go-up

A tiny single-page web app: click a button, watch the number go up. Built as a first "vibe coding" exercise.

The count is persisted in your browser's `localStorage`, so it survives refreshes. Hit Reset to zero it out.

## Running it

Open `index.html` in any modern web browser. That's it — no build step, no dependencies.
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README"
```

---

### Task 7: Final end-to-end manual check

- [ ] **Step 1: Full walkthrough in a fresh browser tab**

Close any open tabs pointing at `index.html`, then open it fresh and verify the full feature set end-to-end:

1. Background is a dark gradient; number `0` (or the previously persisted value) is centered; both buttons visible below.
2. Click the primary button ~10 times — number increments, pulses yellow-gold and scales on each click, and the button has visible hover/press feedback.
3. Refresh the page — the number is preserved.
4. Click Reset — number returns to 0 and shakes.
5. Refresh again — number is still 0.
6. Open devtools → Application → Local Storage → confirm `numberGoUp.count` exists and matches the on-screen value.

- [ ] **Step 2: No commit needed** — this task is verification only.

---

## Done

At this point the spec is fully implemented: single-file HTML app, playful styling, animated increment, reset with shake, and `localStorage` persistence.
