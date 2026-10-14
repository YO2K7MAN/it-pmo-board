# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file, framework-free Kanban board for a fictitious bank's internal IT PMO (demo/training tool). The whole app is `index.html` (markup, `<style>`, `<script>`). There is no build, bundler, package manager, linter, or test suite — run it by opening `index.html` directly in a browser (no server needed).

## Hard constraints (from the original brief)

- Vanilla HTML/CSS/JS only; no frameworks, CDNs, web fonts, or image files. Icons are Unicode/inline SVG.
- No persistence of any kind (no localStorage/sessionStorage/IndexedDB/cookies). Refreshing intentionally resets to the seeded demo data.
- The only network call is FormSubmit's AJAX endpoint (`FORMSUBMIT_ENDPOINT`, top of the `<script>`). It needs one-time email activation; failures must never break the board (always wrapped in try/catch with a warning toast).
- Don't use UOB's real logo/branding; the wordmark is the neutral text "IT PMO".
- CSS uses custom properties for palette/spacing and no `!important`.

## Architecture

- `state = { tasks, filters }` is the single source of truth; `ui` holds transient UI state (`confirmDeleteId`, `draggingId`).
- All card/column content is rendered by `renderBoard()` → `renderCard()` as HTML strings and assigned via `innerHTML`. Every user-supplied value must go through `escapeHtml()`. Don't mutate card DOM directly; change `state`/`ui` and call `renderBoard()`.
- Board events (click, change, drag*) use delegation on `#board` with `data-action` / `data-id` attributes, so they survive re-renders. The "Move ▸" `<select>` on each card is the keyboard fallback to drag-and-drop; the inline "Delete? Yes/No" confirm is driven by `ui.confirmDeleteId`.
- Mutations go through `addTask()`, `moveTask()`, `deleteTask()`; filtering through `applyFilters()` (pure, over `state.tasks`). The summary strip is computed from the unfiltered `state.tasks`; column counts reflect filtered tasks.
- Task IDs come from `nextId()` (`UOB-ITPM-####`, counter `idCounter` starts at the seeded count of 8).
- Form flow (`handleSubmit`): validate → optimistic `addTask()` → close/reset modal → `notifyNewTask()` in parallel.
- Seed dates are generated relative to today via `offsetDate()` so the "Overdue" badge always has examples.
