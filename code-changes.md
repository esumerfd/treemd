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

## Open bugs

### Bug: Inline code in table cells and headers not rendered correctly

**Issue:** [#51 Backticks not rendered correctly](https://github.com/Epistates/treemd/issues/51)

**Root cause:** `ContentBlock::Table` stores cells as `Vec<String>` (no `InlineElement`
support). Inline code formatting is silently dropped when table cells are accumulated.

**Fix required:**
1. Change table cell types in turbovault-parser to `Vec<InlineElement>` per cell
2. Update treemd's `table.rs` renderer to use inline elements instead of plain strings

**Status:** Not yet fixed.
