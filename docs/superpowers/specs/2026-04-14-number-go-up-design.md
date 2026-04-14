# number-go-up — Design

## Purpose

A minimal web page with a button and a counter that goes up on each click. Built as a first "vibe coding" exercise to learn how web apps come together with zero tooling ceremony.

## Stack

- Single `index.html` file containing inline `<style>` and `<script>`.
- No build system, no dependencies, no framework.
- Runs by opening the file directly in any modern browser.

## UI

- Page centered vertically and horizontally, dark playful gradient background.
- Large animated number as the focal point (the current count).
- Primary "Click me" button directly below the number — bright color, hover and press effects.
- Smaller, subdued "Reset" button below the primary button.

## Behavior

- Clicking the primary button increments the count by 1.
- On each increment, the number briefly scales up and color-pulses (CSS transition triggered by toggling a class, then removing it after the animation).
- Clicking Reset sets the count back to 0 and plays a short shake animation on the number for feedback.
- The count is persisted in `localStorage` under the key `numberGoUp.count`. It is loaded on page open and written on every change (increment or reset).

## Files

- `index.html` — the entire application (markup, styles, script).
- `README.md` — one-paragraph description and instructions ("open `index.html` in a browser").

## Testing

Manual only: open the file, click the button several times, refresh the page to confirm persistence, click Reset to confirm it zeroes out and the shake animation plays. No automated test framework — overkill for a single-file static page.

## Out of Scope

- No server, no backend, no network calls.
- No framework, bundler, or package manager.
- No automated tests.
- No multi-user or syncing behavior — persistence is per-browser via `localStorage`.
