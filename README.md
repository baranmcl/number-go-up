# number-go-up

A tiny single-page web app: type a name, click a button, watch the number go up. Built as a vibe-coding exercise.

Each name is its own player with its own counter, all persisted in your browser's `localStorage`. A leaderboard on the right shows the top 10 players and always shows your row, even if you're not in the top 10. Hit Reset to remove your entry from the leaderboard and zero out your counter — other players are untouched.

The leaderboard is local to your browser only. If a friend visits the page on their own device, they get their own empty leaderboard.

## Running it

Open `index.html` in any modern web browser. That's it — no build step, no dependencies.

A hosted copy lives at https://baranmcl.github.io/number-go-up/.
