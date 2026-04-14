# Leaderboard — Design

## Purpose

Add a local, single-browser leaderboard to the number-go-up app. Multiple "players" can compete on the same browser by typing different names into a name input. Each player has their own counter and their own row on the leaderboard. Resetting removes a player from the leaderboard.

This is a local-only feature: it does not introduce a backend, and entries are not shared between browsers or devices.

## Layout

The page becomes a two-column layout:

- **Left column** — the existing experience, shifted left:
  - Name input at the top ("Your name", max 15 characters).
  - Large animated counter.
  - "Click me" primary button.
  - "Reset" secondary button.
- **Right column** — a leaderboard panel:
  - Semi-transparent card with a "Leaderboard" heading.
  - The list of entries below the heading.

On viewports narrower than ~700px, the leaderboard stacks below the counter column so nothing overflows.

## Name input

- A text input labeled "Your name" sits at the top of the left column.
- `maxlength="15"` enforces the character limit at the markup level.
- The trimmed value of the input is the active player's name.
- While the trimmed name is empty, the Click button and Reset button are both disabled.
- Changing the name switches the active player live: the on-screen counter immediately reflects that player's stored count (or 0 if they are new).
- A player only appears on the leaderboard after their first click.

## Leaderboard behavior

- Entries are sorted by `count` descending. Ties are broken by `lastActive` descending (more recently active player ranks higher).
- The top 10 entries are shown. If the active player exists but is not in the top 10, their row is appended below the top 10 in a separate "you" slot, so the active player is always visible.
- The active player's row is visually highlighted (accent border or background tint) wherever it appears.
- Each row shows: rank number, name, count.
- The Reset button removes the active player's entry from the leaderboard entirely *and* zeros the on-screen counter. Other players' entries are untouched.
- Empty-state: when the leaderboard has no entries, the panel shows a muted "No clicks yet" message instead of an empty list.

## Data model

The leaderboard is persisted in two `localStorage` keys:

- **`numberGoUp.leaderboard`** — JSON object mapping `name` → `{ count: number, lastActive: number }`. `lastActive` is a millisecond timestamp from `Date.now()` recorded on the player's most recent click.
- **`numberGoUp.activePlayer`** — the active player's name as a plain string, or absent if no name has been entered. Used to restore the player's identity (and therefore their counter value) on page reload.

The previous version's `numberGoUp.count` key is ignored and intentionally left in place. There is no migration: existing single-player counts do not become leaderboard entries.

## Files

- Modify: `index.html` — all markup, styles, and script changes live here.
- Modify: `README.md` — short note about the leaderboard and the name input.

No new files, no dependencies, no build step.

## Testing

Manual only, consistent with the rest of the project. The implementation plan will list the manual scenarios to walk through (typing a name, clicking, switching names, resetting, refreshing, narrow viewport).

## Out of scope

- No backend, no shared/online leaderboard, no syncing across browsers or devices.
- No name uniqueness enforcement beyond the natural consequence of local storage (two people typing the same name on the same browser share the same row).
- No avatars, no timestamps shown in the UI, no editing or renaming entries after they are created.
- No animations on the leaderboard itself. The counter still pulses on increment and shakes on reset; leaderboard updates appear instantly.
- No migration of the previous `numberGoUp.count` value into the new leaderboard.
