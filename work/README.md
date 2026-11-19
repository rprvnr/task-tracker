# Team Tasks

A one-file task tracker for work. It runs in the browser, on desktop or phone. It can also read your Outlook mail and Wispr Flow meetings and suggest tasks from them.

Everything is in `index.html`. There is nothing to install.

## What it does

- **Projects:** group tasks by project, each with its own colour. Each project has a view with open, done and overdue counts, and time logged.
- **People:** add teammates. Make someone a task's **owner**, mark it **waiting on** someone, or tag the people **involved**. Each person has their own view.
- **Timings:**
  - due date and time
  - an **estimate** (for example 30m or 2h)
  - a **timer** you start and stop, which adds up the time spent. While it runs, a pill at the top shows the time: tap it to open the task, or tap the stop button
  - views show estimated and logged time
- **Views:** Today, Upcoming, All open tasks, Waiting on, Done, plus one view per project and per person.
- **My list:** a small personal list, separate from work tasks. Items sit in **Now**, **Next** or **Later**. Tap an item to edit it, tick it off, or move it with one tap. Clear the Done pile when you like.
- **Light and dark:** the moon/sun button at the bottom of the sidebar switches modes. Settings › Appearance can also match your device.
- **Status:** a segmented control in the task details sets To do, In progress or Done.
- **Suggestions:** scan Outlook and Wispr Flow for action items. Add or dismiss each one, or all at once. Adding or dismissing a single suggestion shows **Undo**.
- **Works on any screen:**
  - On desktop, three columns: sidebar, list and task details.
  - On a phone, the sidebar becomes a menu and task details open full-screen, with a Back button.
  - Light and dark mode follow your system.

## Quick add

Type into the add box and press Enter. Everything except the title is optional.

| Type | Meaning | Example |
|---|---|---|
| `#name` | Project (created if new) | `#launch` |
| `@name` | First one is the owner; any others are involved | `@sam @alex` |
| `!high` / `!!` / `!1` | High priority | |
| `!low` / `!3` | Low priority | |
| `~30m` / `~1.5h` / `~1d` | Estimate (1d = 8h) | |
| `today`, `tomorrow`, `fri`, `2026-12-01`, `12/1` | Due date | |
| `3pm`, `15:30`, `eod` | Due time (eod = 5pm; a time with no date means today) | |

Example: `Send pricing deck #launch @sam fri 3pm ~45m !high`

**Shortcuts:**
- `N` jumps to quick add.
- `/` jumps to search.
- `Esc` closes the open dialog, panel or menu.

## Two ways to run it

### 1. In Claude (recommended, needed for scanning)
Open the page as a Claude artifact. There are two ways to get one:
- Ask whoever set it up to share their artifact link with you. You get your own empty list.
- Or make your own copy: download `index.html`, attach it in a Claude chat, and ask Claude to publish it as an artifact.

Each person also needs:
- The **Microsoft 365** connector, for Outlook.
- The **Wispr Flow** connector, for meetings and notes.

Add both in claude.ai → Settings → Connectors. The first time you press **Scan now**, Claude asks you to allow the page to use them.

In this mode:
- Your tasks are saved in Claude and sync between your devices.
- Your tasks are **private to you**. If you share the page, each person who opens it gets their own empty list. Nobody can see anyone else's tasks.
- Scanning uses **your own** connectors, so it only sees mail and meetings you can already see.

### 2. As a plain file
Open `index.html` in any browser, or host it on an internal web server.
- Tasks are saved in that browser on that device only.
- Scanning is not available, because it needs Claude and the connectors.
- Use Export and Import (in Settings) to move tasks between devices.

## Scanning Outlook and Wispr Flow

Go to **Suggestions** and press **Scan now**. The scan:
1. Reads mail and meetings since the last scan. The first scan looks back 7 days; you can change this in Settings.
2. Skips newsletters and automated senders (no-reply, notifications, marketing).
3. Asks Claude to pick out real action items, with owner, project, due date and time where they're clear.
4. Shows them as suggestions. Each one links back to the email or meeting it came from.

### Duplicates are not suggested again
A scan never suggests something you already have. There are three checks:
1. **Already-read messages are skipped.** Each email, meeting and note is remembered once read. A meeting is read again only if its notes change.
2. **Claude is told what you already have:** your tasks, open suggestions and dismissed suggestions, and is asked not to repeat them.
3. **A final word-match check** removes anything that's close to an existing task, open suggestion or dismissed suggestion, even if it's worded differently.

If you **dismiss** a suggestion, it won't come back.

### Settings
| Setting | What it does |
|---|---|
| Your name | How you appear in emails and notes. This helps Claude work out which tasks are yours. |
| Team or shared mailbox | Also scan a shared mailbox you have access to in Outlook (for example, `team@company.com`). |
| Include shared meetings | Also scan Wispr Flow meetings that teammates shared with you. |
| Suggest tasks for my People list | Off: only suggest your own tasks. On: also suggest tasks owned by teammates on your People list. Use this if you track the team's work. |
| First scan looks back | How many days the first scan reads (1–30). |
| Re-read recent messages | Forgets which messages were read, so the next scan reads them again. Existing tasks are still not suggested. |
| Forget dismissed | Lets dismissed suggestions appear again. |
| Export JSON / CSV, Import JSON | Back up, move or report on your tasks. Inside Claude, exports are saved with the download prompt. To import, paste the contents of an exported JSON file. Import replaces your current list; a file that isn't a task export is rejected and nothing changes. |

## Keyboard
- `/` focuses search.
- `Esc` closes the task details or a dialog.

## Privacy
- Each person's tasks are stored separately, under their own Claude account.
- Mail and meeting text is sent to Claude only during a scan, to find action items. It is not stored. The app keeps only the suggested titles, the sender or meeting name, links back to the source, and the IDs of messages it has read.
- In plain-file mode, nothing leaves your browser.

## Customizing
See [CONTEXT.md](CONTEXT.md) for how the code is organized and where to change things.
