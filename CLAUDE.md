# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A deliberately simple to-do app: plain HTML/CSS/JS, no frameworks, no build step, no tests, no package manager. Keep it that way. Not a git repo.

- `index.html` is the whole app (inline `<style>` and `<script>`). Run it by opening the file in a browser.
- `mockup.html` is a **generated** presentation page: two Samsung Galaxy S23 FE phone frames (light and dark) that each embed a full copy of `index.html` through `<iframe srcdoc>`. Never edit the embedded app code there by hand.

## Architecture (index.html)

- State is a `todos` array of `{ id, text, done, category, due }` (`due` is `YYYY-MM-DD`, `category` is free text). It is persisted to `localStorage` key `todos` as `{ todos: [...] }`; the theme is under key `theme`. Every storage access is wrapped in try/catch. Loading migrates old saves (drops the removed `color`/`next` fields).
- Rendering is a full rebuild: `render()` redraws chips, headers and table rows from state. Event handling is delegated on `<tbody>`, `<thead>` and `#chips`, driven by `data-action`, `data-sort` and `data-filter` attributes. Text is inserted with `textContent` (icon buttons may use `innerHTML` only for the static SVG constant).
- Sorting is by clicking column headers (default: due date ascending, empty values always last, ties keep creation order). There is no sort button.
- Colors are computed at render time, not stored: the due-date bubble (red when overdue or today, then orange, green and blue as the date gets further away, gray when done) and the category bubble (hue hashed from the lowercased name). `normalizeCategory` keeps "work" and "Work" as one category.
- Remaining-time text: days up to 14, then weeks, months and years.
- Theming: CSS variables on `:root`, overridden by `:root[data-theme=dark]`; the default follows `prefers-color-scheme`.
- Layout: the table turns into stacked cards under `max-width: 600px` (CSS grid in the media query). Check both layouts after any markup or CSS change.
- `?demo` loads nine sample tasks with dates relative to today and disables saving; `?theme=light|dark` forces a theme only in demo mode. The mockup relies on both.

## Regenerating mockup.html

After any change to `index.html`, `mockup.html` must be regenerated or its phones show stale code. The generation is: take a mockup template whose phones contain `<iframe src="index.html?demo&theme=light">` and `...theme=dark`, and replace each `src` with `srcdoc="..."`. The srcdoc holds the current `index.html` with `new URLSearchParams(location.search)` replaced by `new URLSearchParams('demo&theme=light')` (or `dark`), then HTML-escaped (`&` to `&amp;`, then `"` to `&quot;`). The srcdoc approach is needed because relative iframe URLs fail in the desktop app's preview pane, which shows local files as `data:` snapshots.

There is no template file in the repo. If you need one, rebuild it from the current `mockup.html` by swapping each `srcdoc` back to the `src` above. Keep the page title and heading starting with "Mockup".

## Gotchas

- In the desktop app's preview pane, `localStorage` does not work on these `data:` pages, so persistence cannot be tested there. Use a normal browser.
- Preview viewport emulation resets after navigation; reset it to desktop after testing.
