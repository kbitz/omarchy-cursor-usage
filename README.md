# Cursor Usage for Omarchy

Adds a Cursor tab to Omarchy's Agents panel (the bot in the bar), next to
Claude Code and Codex, with the same sections they have:

- **Limits**: how much of the plan's monthly allowance is used, in total and
  split into Auto and API, with the time until the billing cycle resets.
- **Tokens by day**: the last week, with today's prompt and session count on
  hover.
- **Tokens by model**: every model used in the last 30 days, with the input,
  output, and cache split on hover.

## Install

```bash
omarchy plugin add https://github.com/kbitz/omarchy-cursor-usage.git --enable
```

Cursor shows up in the Agents panel the next time it refreshes. Opening the
panel triggers one.

## How it works

The Agents panel draws every JSON record in
`~/.local/state/omarchy/agents/usage/`. This plugin is a small service that
runs `cursor-usage` every five minutes, and that script writes `cursor.json`
there in the same format the built-in Claude and Codex collectors use.

The numbers come from cursor.com's dashboard API, the same endpoints
[CodexBar](https://github.com/steipete/CodexBar) uses: limits from
`/api/usage-summary`, and prompts, sessions, and tokens from the last 30 days
of `/api/dashboard/get-filtered-usage-events`. Each usage event is one
prompt and its conversation is the session. Like the Fireworks tab, these are
account-wide numbers, so they include the Cursor app, the CLI, and every
machine you use them on.

Requests are authenticated with the token the Cursor CLI keeps in
`~/.config/cursor/auth.json`, or the Cursor app's token from
`~/.config/Cursor/User/globalStorage/state.vscdb`. The plugin never refreshes
or stores tokens; sign in with `cursor-agent login` or the Cursor app.

The last good answer is cached in
`~/.cache/omarchy/agent-usage/cursor-usage.json`. If a check fails, the panel
keeps showing the cached limits until the billing cycle resets, and the cached
stats for the rest of the day.

Run `./cursor-usage --print` to see the record without writing it.

## When Omarchy adds Cursor itself

Omarchy has built-in Cursor support in review
([omacom/omarchy#14040](https://github.com/omacom/omarchy/pull/14040)).
It writes the same `cursor.json`, so once an update puts
`omarchy-agent-usage-cursor` on your system, this plugin stops writing the
record and stops deleting it on removal. The built-in version takes over, and
you can remove this plugin whenever you like.

Until then the tab uses the panel's default glyph: the panel only loads marks
from its own built-in folder, and the Cursor mark arrives with that change.

## Hide or remove

```bash
omarchy plugin disable kbitz.cursor-usage
omarchy plugin remove kbitz.cursor-usage
```

Either one deletes `cursor.json`, so the Cursor tab disappears with it.
