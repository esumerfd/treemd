# Code Changes

Tracks bugs fixed and corresponding upstream patch files.

---

## turbovault-parser

### Bug: Inline code in headings and blockquotes not rendered correctly

**Issue:** [#51 Backticks not rendered correctly](https://github.com/Epistates/treemd/issues/51)

**Root cause:** In `blocks.rs`, the `Event::Code(text)` handler in `process_event` called
`add_inline_text` unconditionally, which always wrote to `paragraph_buffer`/`inline_buffer`.
When fired inside a heading or blockquote, the code content went to the wrong buffers:

- **Headings** — code text was missing from `heading_inline`, causing it to not render.
  The leaked text was then prepended to the next paragraph's content.
- **Blockquotes** — code text was missing from `blockquote_buffer`, leaving a gap in the
  rendered output. The leaked text contaminated the next block.

**Fix:** Route `Event::Code` based on context — to heading buffers when `in_heading`,
re-emitting with backtick delimiters into `blockquote_buffer` when `in_blockquote`.

**Patch file:** `turbovault-parser-heading-blockquote-inline-code.patch`

**Status:** Patch generated, pending application and publish of turbovault-parser 1.2.8.
Vendored fix active in `crates/turbovault-parser` via `[patch.crates-io]` in `Cargo.toml`.

---

## turbovault-parser

### Bug: Inline code in table cells and headers not rendered correctly

**Issue:** [#51 Backticks not rendered correctly](https://github.com/Epistates/treemd/issues/51)

**Root cause:** `ContentBlock::Table` stores cells as `Vec<String>`. When `Event::Code`
fired inside a table, it fell through to the default branch writing to `paragraph_buffer`
without backtick delimiters, so inline code formatting was silently dropped.

**Fix:**
1. Added `else if state.in_table` branch to `Event::Code` in `blocks.rs` to re-emit
   backtick delimiters into `paragraph_buffer` (same approach as blockquote fix).
2. Updated `src/tui/ui/table.rs` `render_table_row()` to call `format_inline_markdown()`
   on cells containing backticks, producing styled spans instead of a single plain span.

**Patch file:** `turbovault-parser-table-inline-code.patch`

**Status:** Patch generated, pending application and publish of turbovault-parser 1.2.8.
Vendored fix active in `crates/turbovault-parser` via `[patch.crates-io]` in `Cargo.toml`.
