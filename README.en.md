# dsh-session-manager

English | [中文](README.md)

<p align="center">
  <a href="./LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-3167E3?style=flat-square"></a>
  <img alt="DeepSeek Harness" src="https://img.shields.io/badge/DeepSeek%20Harness-0.1.0--rc.6-3167E3?style=flat-square">
  <img alt="Version" src="https://img.shields.io/badge/dsh--session--manager-v0.1.8-3167E3?style=flat-square">
</p>

`dsh-session-manager` is a comprehensive DSH session management plugin providing full session management capabilities: delete (with trash/restore/purge), restore archived sessions, activity stats, continue/pause, fork to a new chat, unread markers, and open log folders. Plus a global context compaction threshold setting.

**Clean Version**: Removed all button entries from the conversation header; all session management features are accessible through Settings for a cleaner interface.

<sub><span style="opacity:.6">Based on dsh-session-manager, maintained by Xinyu-lumos</span></sub>

<sub><span style="opacity:.6">If you find it useful, consider giving it a ⭐ Star. Thanks for your support!</span></sub>

## Features

- A dedicated **Session Manager** section in Settings (a settings section, sibling to Notifications)
- Lists all sessions (title / working directory); **archived sessions** are grouped in a collapsible area at the bottom with a **one-click Restore** back to the list
- **Trash**: deleted sessions move to the trash (keeps the most recent 10, the oldest is purged automatically), with **Restore** and **Delete permanently** actions
- **Stats**: expand any session to see recent activity (turns / user messages / assistant messages / tool calls / activity window)
- **Continue session**: open a session and close the panel; **Pause**: stop a running session's current turn
- **Unread / read**: a status dot next to each session's title — blue for manually marked unread, amber for the official waiting-for-input state, green for the official completion reminder, spinner while running; clicking an official dot marks it read **in place** (no navigation), clicking the blue dot clears the unread, opening a session auto-reads it; the official sidebar shows a matching blue unread dot on the session row
- **Fork into a new chat**: one click forks a child session (official `sessions.fork`) and opens it
- **Folder**: reveal the session's log directory in the system file manager
- **Workspace management**: sessions are grouped by workspace, sorted by last use within each group (toggle newest/oldest first); drag a workspace title to reorder (insert before/after, swap on the title, drag to the bottom to append); hovering a title shows **Move to top / Rename / Delete** buttons (delete follows the official definition: it only removes the workspace from the list — the folder and session logs are kept, and its sessions appear under Ungrouped)
- **Context compaction threshold** (General settings): set at what fraction of the 1M-token model window the conversation context auto-compacts (17%–90%), keeping the most recent 16% verbatim; applies **globally to all agent presets** (immediate on save + persisted + auto-applied on restart)
- Delete restriction: only sessions **currently thinking** are protected; an open-but-idle session can be deleted
- Subagent sessions can be deleted when not running: even orphaned ones (whose parent session is already deleted) can be cleaned up directly from Session Manager
- UI language follows the page language (Chinese / English)
- **Clean Interface**: Removed all session management buttons from the conversation header; unified access through Settings for a cleaner conversation interface

## Screenshots

The Settings "Session Manager" section (workspace groups, row actions and trash):

![Session Manager settings section](assets/settings-section.png)

The "Context compaction threshold" in General settings (17%–90% with slider scale):

![Context compaction threshold](assets/general-settings.png)

## Install

### From GitHub

```sh
dsh plugin --profile web add 'https://github.com/Xinyu-lumos/dsh-session-manager.git'
```

### From a local directory

```sh
dsh plugin --profile web add /absolute/path/to/dsh-session-manager
```

### From a tarball

```sh
pnpm pack
dsh plugin --profile web add /absolute/path/to/dsh-session-manager-0.1.8.tgz
```

After installing, **restart** `dsh web` (the host plugin and the served client bundle load at startup).

## Usage

### Settings section

1. Open **Settings** (the gear icon at the bottom of the sidebar)
2. A dedicated **Session Manager** section appears in the settings left navigation — click it
3. The main list shows unarchived sessions; the **Archived sessions** collapsible area at the bottom lets you view, **restore**, or delete archived sessions
4. Deleting moves a session to the **Trash** collapsible area (keeps the most recent 10)
5. In the trash you can **Restore** (back to the list) or **Delete permanently** (irreversible)
6. Per-row actions: **Continue session** (open and enter), **Pause** (stop the running turn), **Stats** (expand recent activity), **Folder** (reveal the log directory), **Delete**
7. Workspace title actions (shown on hover): **Move to top**, **Rename**, **Delete** (red, with a confirmation dialog)
8. Drag a workspace title to reorder: drop above/below another workspace to insert, drop on a title to swap, drag to the very bottom to append
9. The sort toggle (newest first / oldest first) switches the session order inside each group

### General settings: context compaction threshold

1. Open **Settings** → **General**
2. Find "Context compaction threshold": slider / input for 17%–90%
3. Saving applies immediately (including already-open sessions); the value applies globally to all agent presets and survives restarts

### Unread / read status dots

The dot next to a session's title shows one of four states: **blue** = manually marked unread, **amber** = official waiting-for-input, **green** = official completion reminder, **spinner** = running. Clicking an amber/green dot marks it read **in place** (clears the official reminder without navigating); clicking the blue dot clears the unread; clicking the empty spot marks it unread; opening the session auto-reads it. The official sidebar mirrors the blue unread dot on the matching session row (matched by title text — sessions with duplicate titles share the dot).

## How it works

| Layer | Implementation |
|---|---|
| Host | `src/index.ts` registers 7 routes: `POST /delete` (archive + move non-live session files to the trash + record the entry), `POST /restore` (move files back + unarchive + drop the entry), `POST /purge` (clear trash and original files + drop the entry), `GET /trash` (trash listing), `POST /open-folder` (reveal the log directory), `POST /pause` (pause a running session), and `GET|POST /compaction-threshold` (read/write the global compaction threshold). It resolves sessions via `ctx.sessionPersistence`, archives/unarchives through `ctx.workspaceRegistry`, and persists trash entries, the archive set and the threshold via `ctx.storageDomain`; `ctx.agents` detects running sessions and refuses to delete them |
| Client | `src/client/index.ts` registers the dedicated section through the official `settings.section` slot, lists sessions (with the archived group) from the `useSessions` / `useWorkspaces` standard feeds, and calls the host routes to delete/restore/purge; the drawer subscribes to the live session list via `sessions.list` (an ObservableSnapshot); removed session ids are remembered in browser localStorage so a live session does not "resurrect" after refresh; **Removed `conversation.session.header.utilities` slot registration**, no buttons shown in conversation header |

- **Unread mechanism**: the manual unread set lives in browser localStorage under the shared key `dsh.session-unread.v1` (format `{version:1, ids:[]}` — interoperable with other session-manager plugins); the official dots (amber/green/spinner) are driven by the official `SessionSummary` fields `pendingInteraction` / `completed` / `running`, and clicking one marks it read in place by clearing the official reminder (no session open); the sidebar blue dots are decorated onto the official tree rows by a MutationObserver (official row elements carry no session-id attribute, so rows are matched by title text)
- **Global threshold**: stored in the storage domain (`dsh_delete_session` → `thresholdRatio`) and in the current agent preset's `agent.cordis.yml`; the host writes the threshold into every preset's compaction-engine config in an `agent/pre-step` hook, so it applies to all presets uniformly and survives restarts
- Deletion goes through the official archive channel first: the sidebar hides the session immediately
- Trash entries persist in the DSH storage domain (`~/.dsh/storages/dsh_delete_session.json`); files live in `~/.dsh/dsh-delete-session-trash/`
- Workspace accounting (`sessionIds` slots / the archive set) is reconciled automatically on the next startup when the registry rebuilds its header index — no manual file editing
- No system-prompt changes, no new model-facing tools: zero impact on tokens and model behavior

## Limitations

- **Running sessions cannot be deleted** (button disabled and the host refuses); with multiple tabs, stop the session elsewhere first
- Subagent sessions can be deleted when not running — including orphaned ones left behind by a deleted parent session, so no residue stays forever
- A live session (opened in this process) has its in-memory state cleaned up by DSH on restart; deleted ids are recorded in browser localStorage so they do not reappear after a refresh
- Sidebar unread dots are matched by title text: sessions with duplicate titles share the same dot (the drawer is unaffected — it marks by real session id)

## Compatibility

Current version targets DSH `0.1.0-rc.6` (depends on the `settings.section` / `settings.general.item` slots and the `ctx.sessionPersistence` / `ctx.workspaceRegistry` / `ctx.agents` / `ctx.storageDomain` / `ctx.agentPresets` services). If slots or service APIs change in a future DSH version, the plugin needs a matching update.

## Development

```sh
pnpm install        # installs dependencies (@deepseek-ai packages are linked local dev dependencies)
pnpm run check      # typecheck + test + build
```

`lib/` holds the committed build artifacts: rebuild and commit `lib/` with every source change.

## Acknowledgments

Based on [dsh-session-manager](https://github.com/dream12347/dsh-session-manager).

## License

MIT
