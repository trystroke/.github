# Try Stroke

Try Stroke is a self-hosted, keyboard-first mail client for Gmail and Microsoft mail. The mailbox lives on a host you run. The browser is a triage screen for that host: inbox, an open thread, and compose.

An action is recorded before it reaches Gmail or Microsoft. Archive, read, star, trash, send, and snooze are appended to an ordered log on that thread. The screen and the local index update together. A worker later replays the log onto the provider. Closing the tab is safe, because the action is already on the host.

## Triage

The thread list is windowed. Keys act on the selected thread:

| Key | Action |
| --- | --- |
| `j` / `k` | Move through the list |
| `e` | Archive |
| `u` | Mark read or unread |
| `s` | Star or unstar |
| `c` | Compose |
| `z` | Undo |
| `h` | Snooze |
| `Cmd-K` | Open the command palette |

Undo covers the last action, including send. A send waits through a short undo window, about ten seconds, before it is handed to the provider. Undo during that window removes the send. After the window, the worker delivers it. The host stays up, so closing the browser does not drop a send that has already been accepted.

Snooze stays on the host. The thread leaves the inbox until the wake time, then returns. Gmail and Microsoft never receive a snooze.

## How an action lands

1. The keystroke updates the screen immediately.
2. The same action is stored on the thread’s log, and the index updates in that same step.
3. The worker replays pending actions onto Gmail or Microsoft. Each action is idempotent, and the provider result is stored so a retry does not send twice.
4. When new mail arrives, the host stores the provider’s snapshot, then folds pending actions back on top. A poll cannot flicker an archived thread back into the inbox.

## Accounts

Connect Gmail, Microsoft 365, or Outlook. You bring your own OAuth app: a Google OAuth client, and an Azure app registration. Mailbox sign-in is separate from the admin session on the host.

Gmail archive removes the inbox label. Star uses the starred label. Read uses the unread label.

Microsoft archive moves the message to the Archive folder. Star sets the flag. Read sets the read state. A move into a folder the message is already in counts as success.

New mail is polled. If a provider rate-limits an account, only that account backs off.

## Search

Search runs on the host, over subject and body:

- A bare word matches that text. `"a quoted phrase"` matches the phrase.
- `from:` matches the sender. Quote the value when it contains spaces.
- `in:` filters by label. `in:inbox`, `in:unread`, `in:starred`, `in:trash`, `in:sent`, and `in:snoozed` are the built-in labels. Any other value is a label id.
- `is:unread`, `is:starred`, and `is:snoozed` are the same labels, written as states.
- Adjacent terms are AND. `OR` and parentheses group alternatives. `and` and `or` are operators in any case; quote them to search for the word.

`dinner (from:ada OR from:grace)` finds dinner from either sender. `in:inbox is:unread` is the unread inbox.

## On the host

One process serves the app and the sync loop. Mail is a single SQLite file: accounts, threads, messages, labels, the action log, and each account’s sync cursor. Message text is indexed for search. OAuth refresh tokens are encrypted at rest.

The browser keeps no mailbox of its own. Try Stroke expects the host to be reachable.

## Status

The mail model, the local index, search, and the Gmail and Microsoft adapters are in place. The HTTP API, the sync worker, and the triage screen are still being built, so there is no install path yet.
