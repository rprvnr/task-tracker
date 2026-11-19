# Team Tasks: technical context

This file is for anyone changing the code, and for an AI assistant you ask to help. Read README.md first for what the app does.

## Overview
- **One file:** `index.html` holds the HTML, CSS and plain JavaScript (ES5 style). There are no dependencies and no build step.
- **Two modes:**
  - **Claude mode:** the page detects `window.claude`. It stores data with the artifact `db` capability, reads mail and meetings with `mcp`, and finds action items with `sample`.
  - **Local mode:** if `window.claude` is not there after about 2 s, the page saves to `localStorage` under the key `team-tasks-state`. Scanning is turned off.
- The app has a single `state` object. Every change calls `changed()`, which runs `normalize()`, then `render()`, then `save()`.

## Layout
- CSS grid with three columns: sidebar (248px), main list, and detail panel (400px).
- At 1180px or narrower, the detail panel becomes an overlay.
- At 820px or narrower, the sidebar becomes a drawer. The `.app.nav-open` class opens it, and `#scrim` covers the page behind it.
- Colours are CSS variables at the top of the file. Dark colours apply under `:root[data-theme="dark"]`, or under `prefers-color-scheme: dark` unless `data-theme="light"` is set.
- `applyTheme()` sets `data-theme` from `state.settings.theme` (`system`, `light` or `dark`) and mirrors it to localStorage `team-tasks-theme`, so the page opens in the right mode before data loads.
- The personal list is `state.personal`: `{ id, text, sec: "now"|"next"|"later", doneAt, created }`. `renderPersonal()` draws it. Rows carry `data-ps`; actions are `pToggle`, `pMove`, `pDel`, `pClear`. Text is a contenteditable `.ptext`; emptying it deletes the item.

## Data model
```js
state = {
  v: 1, nextId: 1,
  tasks: [{
    id: "t12", title, notes,
    status: "todo" | "doing" | "done",
    priority: "high" | "normal" | "low",
    project: "p3" | "",            // project id
    owner: "me" | "u4",            // person id
    waiting: "u5" | "",            // waiting on this person
    people: ["u6"],                // people involved
    due: "YYYY-MM-DD" | "", time: "HH:MM" | "",
    estimate: 90,                  // minutes
    spent: 3600,                   // seconds already logged
    timerStart: ISO | null,        // set while the timer runs
    created: ISO, doneAt: ISO | null,
    source: { kind, from, link }   // set when the task came from a suggestion
  }],
  projects: [{ id: "p3", name, color }],
  people:   [{ id: "u4", name, email }],
  inbox: {
    items: [...],          // open suggestions
    seen: ["m:<id>", "w:<id>:<modified_at>", "n:<id>:<modified_at>"],  // keeps the last 4000
    dismissed: ["title"],  // keeps the last 600
    lastScanMail, lastScanFlow, note
  },
  settings: { me, teamMailbox, includeShared, forTeam, firstDays }
}
```
- `normalize()` fixes missing fields. It also clears links to deleted projects and people. Run it after any load or import.
- Views are strings: `today`, `upcoming`, `all`, `waiting`, `done`, `inbox`, `project:<id>` and `person:<id>`. `groupsFor(view)` builds the sections for each view.
- `parseQuick(text)` handles the quick-add syntax. README.md lists the syntax.
- **Look:** Apple-style. Icons are inline SVGs in the `ICON` map (no emoji). Nav items use coloured icon tiles. Status uses a segmented control (`.seg`, `data-act="setStatus"`, with `aria-pressed`).
- **Undo:** single accept and dismiss save a snapshot; the toast's Undo button (`#toastUndo`) restores it.
- **Export:** uses `claude.use('downloads').save({filename, data})` when available, else a normal browser download. **Import** is a paste box in a dialog; the JSON must contain a `tasks` array.
- **Links** open in the same tab (no `target=_blank`), since the page runs in a frame.

## Storage in Claude mode
- **Document path:** `data/users/<user.id()>/state`. Each user has their own document, so nobody can see anyone else's tasks.
- **Profile:** `user.me()` gives the person's display name. The app uses it as the default for "Your name".
- **Saving:** `save()` waits 300 ms, then `flush()` writes the whole state with `doc.set()`. Only one write runs at a time; any later change is written next.
- **Incoming changes:** `onSnapshot` sends changes in to `applyRemote()`. It ignores the app's own pending writes, and it ignores any update while a local write is waiting or in progress.
- **Slow first connection:**
  - Until the first snapshot arrives from the server (not from cache), the app holds back all writes. This stops an empty or stale local state from overwriting saved data.
  - Any edits made in that time are combined into the saved data by `mergeHeld()`. Tasks created since the page loaded are added on top, with new ids. Their project and people links are matched again by name.
- **Write errors:**
  - `unavailable`: retried once.
  - `permission_denied`, `not_granted`, `revoked`, `invalid_argument`: saving stops and the page shows "Can't save here".
  - `quota_exceeded`: the page shows "Storage full".

## Scanning (the `scan()` function)
The scan runs in this order:
1. **Fetch new items.** Microsoft 365 and Wispr Flow are read in parallel. Each source has its own time window: `inbox.lastScanMail` and `inbox.lastScanFlow`. The first scan uses `firstDays`.
   - **Outlook:** `outlook_email_search` with `{query:"*", afterDateTime, limit:25, offset, mailboxOwnerEmail?}`, up to 3 pages. The shared mailbox gets up to 2 pages. The result is a list of JSON objects in text blocks, and the last one is `{moreResults, nextOffset}`. `parseMulti()` and `objsOf()` read it.
   - **Wispr Flow:** `search_meetings {since, limit:20}` and `search_scratchpad_notes {since, limit:15}`. For up to 8 new meetings, the scan then calls `get_meeting {meeting_id, view_content:{char_limit:1}}`. That returns the AI summary, and `nextSteps()` cuts out its "Next Steps" or "Action Items" section. Shared meetings are skipped unless `includeShared` is on.
2. **Skip items already read.** The keys are `m:` plus the message id, `w:` plus the meeting id and `modified_at`, and `n:` plus the note id and `modified_at`. A meeting whose notes change gets a new key, so it is read again.
3. **Filter automated mail.** The `AUTOMATED` regex skips no-reply addresses, notifications, newsletters and marketing.
4. **Ask Claude.** `sample.json(prompt)` gets at most 80 source items. The prompt includes:
   - today's date and "Your name"
   - the People list (when `forTeam` is on)
   - the project names in use
   - up to 250 existing tasks, all pending suggestions and the last 120 dismissed titles, with an instruction not to repeat them

   Claude returns up to 15 objects of the form `{title, ref, owner, project, due, time}`.
5. **Remove duplicates.** `isDuplicate(title, link)` uses a word-overlap score, `similar()`:
   - words are lower-cased, stopwords removed and suffixes trimmed
   - the score is the Jaccard index, or for titles of 3 or more words, containment × 0.85, whichever is higher

   A score of 0.6 or more against any task, pending suggestion or dismissed title counts as a duplicate. So does a score of 0.35 or more against a task from the same source link.
6. **Store the results.** The suggestions are saved, the seen keys added and the scan times moved forward. A source's scan time moves forward only if that source was read without an error.

Each suggestion stores its project and owner **as names**. Accepting it (`acceptSuggestion()`) turns those names into ids with `findProject(name, true)` and `findPerson(name, true)`, creating the project or person if needed. Dismissing it adds its title to `inbox.dismissed`.

### Connector errors
`mcpMessage()` turns connector error codes into messages for the user:

| Code | What it means |
|---|---|
| `needs_reauth` | The connector needs to be reconnected. |
| `server_not_connected`, `server_not_found` | The connector needs to be added. |
| `selection_required` | The user needs to choose which connector to use. |
| `approval_required`, `consent_required`, `not_in_manifest` | The user needs to allow the page to use the connector. |
| `blocked_by_policy` | The organization doesn't allow this tool. |
| `not_granted` | The page is not running in Claude. |
| `server_unavailable`, `rate_limited` | The connector didn't answer; try again. |

Retryable errors are tried once more after `retryAfterMs`. If one source fails, the others still finish, and the scan note says what couldn't be read.

### Capabilities needed when publishing as a Claude artifact
- `db`
- `user` (profile)
- `sample`
- `mcp`, with these tools:
  - Microsoft 365 → `outlook_email_search`
  - Wispr Flow → `search_meetings`, `get_meeting`, `search_scratchpad_notes`

## Common changes
| To change | Look at |
|---|---|
| Colours or theme | CSS variables at the top of the file; `COLORS` for project colours |
| Quick-add syntax | `parseQuick()` and `quickPreview()` |
| What counts as "Today" or overdue | `isToday()`, `isOverdue()` |
| A new view | `navItem` calls in `renderNav()`, and `groupsFor()` |
| How strict the duplicate check is | the `0.6` and `0.35` values in `isDuplicate()`, and the `STOP` word list |
| Which senders are skipped | `AUTOMATED` |
| What Claude looks for | the prompt text inside `scan()` |
| Another source, for example Teams chats | add a fetch beside `fetchMail` and `fetchFlow` that returns `{key, source, from, subject, text, link}` items, add it to the `Promise.allSettled` in `scan()`, and add the tool to the capability list |
| A new task field | add a default in `normalize()`, an input in `renderDetail()`, and a column in `toCsv()` |

## Testing
The app was tested with Playwright in a headless browser:
- Local mode: quick-add parsing, saving after reload, settings.
- A mock `window.claude` with fake `db`, `user`, `mcp` and `sample`. These tests checked:
  - the per-user document path
  - newsletter filtering
  - that a rescan skips messages already read without calling Claude
  - that reworded duplicates are removed, and dismissed items stay dismissed
  - that accepted tasks keep their source link and project
  - that edits made during a slow connection are kept

To repeat these tests, put a fake `window.claude` in place with `page.addInitScript` before loading `index.html`.
