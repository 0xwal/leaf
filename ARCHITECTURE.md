# Architecture

`leaf` is a terminal Markdown previewer built around a small set of focused modules:

- `src/main.rs`
  - entrypoint
  - loads CLI options
  - reads the initial document or opens the file picker
  - initializes terminal + syntax/theme assets

- `src/app.rs`
  - central runtime state
  - document content, TOC, search state, watch state
  - theme picker + file picker state
  - cache invalidation and document replacement helpers

- `src/markdown.rs`
  - Markdown parsing and render preparation
  - heading, TOC, list, table, blockquote, code block rendering
  - width-aware wrapping helpers

- `src/render.rs`
  - draws the TUI with `ratatui`
  - main content, TOC, status bar
  - modal rendering for help, theme picker, and file picker

- `src/runtime.rs`
  - event loop
  - keyboard/mouse handling
  - watch polling
  - resize-driven render width synchronization

- `src/theme.rs`
  - UI and Markdown theme presets
  - active theme preset selection
  - syntect theme mapping

- `src/cli.rs`
  - command-line parsing
  - usage/version text

- `src/terminal.rs`
  - raw mode / alternate screen lifecycle
  - terminal restore guarantees

- `src/tests.rs`
  - regression tests for rendering and state behavior

## Execution flow

1. `main.rs` parses CLI options.
2. A document is loaded from:
   - a file argument, or
   - `stdin`, or
   - the file picker if no input is provided interactively.
3. `markdown.rs` parses the source into rendered lines + TOC.
4. `App` stores the state and caches.
5. `runtime.rs` runs the event loop.
6. `render.rs` draws each frame from `App`.

## Important state transitions

- document reload / open:
  - source changes
  - rendered lines and TOC are rebuilt
  - caches are refreshed

- resize:
  - effective render width is recomputed
  - Markdown is reparsed width-aware

- theme preview:
  - previewed content is reparsed and cached per preset
  - `Esc` restores the original theme

- search:
  - query state lives in `App`
  - active match drives highlight + scroll position

## Current hotspots

- `src/app.rs` still centralizes many responsibilities
- `src/markdown.rs` is the densest module and the main future split candidate
- `src/render.rs` is growing as more modal UI is added
