# Claude Usage Notifier

Get notified through the Claude desktop app when your Claude plan usage limits
are running low — both the rolling **session** (5-hour) limit and the
**weekly** limit.

## How it works

Unlike turn-completion notifications, plan usage data isn't available to a
plain shell hook — it's only exposed to a Claude session running inside the
Desktop app, via its built-in `get_usage` tool. So this project is a
**Claude Code scheduled task** (not a script): a background Claude session
that wakes up every few minutes, checks your usage, and pushes a notification
through the Windows Claude app when it's warranted.

The task is named `usage-limit-notifier` and lives in
`~/.claude/scheduled-tasks/usage-limit-notifier/SKILL.md`. It runs only while
the Claude Desktop app is open.

Each run:

1. Reads [`config.json`](config.json) for the configurable session thresholds
2. Calls `get_usage` to read the account's plan limits:
   - the **5-hour limit** window = "session usage"
   - the **Weekly** window(s) = "weekly usage"
3. **Session usage**: the first time percent-used crosses each threshold in
   `sessionThresholds` (currently `[25, 50, 75, 90]`), sends one notification
   for that threshold. Each threshold only fires once per 5-hour window.
   **100% always notifies too**, regardless of what's in `config.json` — it
   isn't removable via the config.
4. **Weekly usage summary**: when the 5-hour window's reset time changes from
   the last check (i.e. a session just ended and a new one began), sends a
   notification with the current weekly usage — a summary "at the end of
   every session," as requested.
5. **Weekly limit reached**: independent of the above, if any weekly window
   itself hits 100%, sends a notification immediately — it doesn't wait for
   the next session to end.
6. Saves the updated state back to `state.json`.

All of this runs unattended in the background on its own schedule.

## Customize

Edit [`config.json`](config.json):

```json
{
  "sessionThresholds": [25, 50, 75, 90]
}
```

Add, remove, or change percentages freely. Changes take effect on the next
scheduled run — no restart needed. Note: raising or adding a lower threshold
that the current session has already passed will fire a one-time catch-up
notification for it on the next run, since it hasn't been notified yet this
window.

## Check on it

- List scheduled tasks and their last run in the Claude app's task list, or
  ask Claude to "list scheduled tasks."
- `state.json` shows what's already been notified for the current window.

## Uninstall

Ask Claude to delete the `usage-limit-notifier` scheduled task, or remove
`~/.claude/scheduled-tasks/usage-limit-notifier/` by hand.
