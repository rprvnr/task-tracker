# Task Tracker

A one-file, mobile-first daily task tracker. Open `index.html` in a browser and it works. There's nothing to install.

## Features
- **Sections:** Now (this week), Next (next few weeks), Later, Done, plus a separate shopping list.
- **Filters:** Personal, Work, Shop and Daily tabs, with tags like Home, Errands and Goals.
- **Today star:** tap ☆ to mark a task for today.
- **Swipe:** swipe right to complete a task, or left to push it to tomorrow. You can also tap a task and use the **Tomorrow →** button.
- **Quick add:** type plain language, like "call dentist tomorrow #personal" or "water plants every week".
- **Repeating tasks** and **daily routines** with tap counters (for example, glasses of water).
- **Drag** tasks to reorder them, with undo for mistakes.

## Saving
- **Opened as a plain file or web page:** your list is saved in that browser, on that device only.
- **Opened as a Claude artifact:** your list syncs through Claude. There's also an optional **Scan inbox** button that uses your Microsoft 365 (Outlook and Teams) and Wispr Flow connectors to suggest action items.

## Customizing
Everything is in `index.html`:
- The starting example tasks are in the `<script id="state">` block.
- Colors are CSS variables at the top of the file.
