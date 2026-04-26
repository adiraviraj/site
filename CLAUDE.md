# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

This is a **single-file static site**. Everything — markup, CSS, JavaScript, and post content — lives in [index.html](index.html). There is no build step, no package manager, no dependencies, no test suite. Assets live in [image/](image/).

To preview locally, open `index.html` in a browser (or `python3 -m http.server` from the repo root if you need a proper origin for anything). The git remote is `github.com/adiraviraj/site`; there is no configured CI or deployment pipeline visible in the repo.

## The UI metaphor

The site renders as a faux Windows 95 desktop and should be kept consistent with that aesthetic when making changes:

- A draggable, maximizable window holds a menu bar, content area, and status bar.
- Minimize triggers a bouncing "ADI.EXE" screensaver + taskbar restore button (`minimize()` / `restore()`).
- Close runs a fake shutdown → BIOS boot sequence before restoring (`shutdown()` / `runShutdownSequence()` / `runBootSequence()`).
- Color tokens (`--blue-title`, `--green-accent`, `--orange-accent`, `--win-grey`, etc.) are defined in `:root` and referenced throughout. Prefer these over raw hex values when adding styles.

## Content model (this is the non-obvious part)

Posts are **not** markdown files — they are JavaScript object literals inside the `<script>` block of `index.html`. Three top-level arrays drive the content:

- `posts` — array of blog entries. Each entry: `{ id, size, title, date, excerpt, body }`. `id` is used as the anchor for `openPost(id)`; `size` controls the stream card style (`"big"` is the currently-used value).
- `fragments` — standalone text snippets interleaved between posts on the stream view.
- `aboutText` — array of paragraphs rendered by `showAbout()` for the About.txt view.

`post.body` is an **array of blocks** rendered by `renderBlock()`. Block forms:

- Plain string → rendered as a `<p>`. **Inline HTML is allowed and used** (e.g. `<a>`, `<em>`) — strings are injected via `innerHTML`, so treat them as trusted authored content.
- `{ type: "heading", text }` → section heading
- `{ type: "divider" }` → `<hr>`
- `{ type: "quote", text }` → `<blockquote>`
- `{ type: "disclaimer", text }` → styled note box
- `{ type: "image", src, caption }` → image with optional caption (`src` is typically `./image/foo.jpg`)
- `{ type: "video", src, caption, isEmbed }` → `<iframe>` when `isEmbed`, otherwise `<video controls>`

### Adding a new post — watch out for this

`buildStream()` currently hardcodes the stream layout: `fragments[0]`, `posts[0]`, `posts[1]`, `fragments[1]`. Appending a new object to `posts` will **not** make it appear in the stream — you must also edit `buildStream()` to insert it. The status bar entry count (`status-count`) is driven off `posts.length` and updates automatically.

`readTime()` estimates reading time at 200 wpm by flattening all string blocks plus any block with a `.text` field, then stripping inline HTML.

## View switching

Two views share the content area and are toggled by swapping `display`:

- `stream-view` — the list/feed, populated once by `buildStream()` on load.
- `reader-view` — populated on demand by `openPost(id)` or `showAbout()`.

`switchToReader()` / `showStream()` also swap the title-bar text, the "All Posts" ↔ "← Back" menu item, and the `status-mode` label. Keep these three in sync when adding new views.
