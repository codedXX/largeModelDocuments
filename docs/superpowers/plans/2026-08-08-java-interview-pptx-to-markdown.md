# Java Interview PPTX to Markdown Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert `JavaInterview/02-Redis篇.pptx` into a faithful, well-structured Markdown document with all embedded images stored in `JavaInterviewImages`.

**Architecture:** Parse the PPTX package as OOXML, preserving slide order and ordering slide elements by their on-slide coordinates. Render the original deck for visual comparison, then generate Markdown headings, lists, emphasis, code blocks, tables, and image references without rewriting the source wording.

**Tech Stack:** Python standard library (`zipfile`, `xml.etree.ElementTree`), PowerPoint rendering when available, Markdown validation scripts.

## Global Constraints

- Preserve the PPTX wording; do not add, remove, paraphrase, or correct source content.
- Use Markdown structure and emphasis only to improve readability.
- Export all PPTX images into `JavaInterviewImages`.
- Every Markdown image reference must start with `../JavaInterviewImages/`.

---

### Task 1: Inspect Source Deck

**Files:**
- Read: `JavaInterview/02-Redis篇.pptx`
- Create: temporary extraction and inspection artifacts outside final deliverables

- [ ] Inspect slide order, shape coordinates, text runs, paragraph levels, and image relationships.
- [ ] Render every source slide for visual comparison.
- [ ] Record slide titles and content-bearing elements in reading order.

### Task 2: Generate Markdown and Images

**Files:**
- Create: `JavaInterview/02-Redis篇.md`
- Create: `JavaInterviewImages/*`

- [ ] Export referenced media with stable slide-based filenames.
- [ ] Convert titles and body hierarchy to Markdown headings and lists.
- [ ] Preserve code-like text as fenced code blocks and highlight source-emphasized text with Markdown bold.
- [ ] Insert each image at its corresponding position using `../JavaInterviewImages/<filename>`.

### Task 3: Verify Fidelity

**Files:**
- Verify: `JavaInterview/02-Redis篇.md`
- Verify: `JavaInterviewImages/*`

- [ ] Compare each source slide render with the corresponding Markdown section.
- [ ] Verify every PPTX text string is represented in the Markdown, excluding repeated template chrome and slide numbers.
- [ ] Verify every image reference resolves to an exported file and no exported image is orphaned.
- [ ] Check that the source PPTX remains unchanged.
