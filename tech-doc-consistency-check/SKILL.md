---
name: tech-doc-consistency-check
description: >
  This skill should be used when the user asks to "check consistency", "audit the doc",
  "review this document", or "check for errors" on a Markdown technical document.
  Covers section numbering, TOC correctness, internal hyperlink validity, HTML anchor
  validity, spelling (tech-aware), and basic prose grammar.
license: CC0-1.0
metadata:
  author: Vincent Yin
  version: "1.0.0"
---

# Tech Doc Consistency Checker

Run all checks below in order. Fix every issue found automatically unless the fix requires a judgment call — in that case, report it to the user and ask.

---

## Check 1 — Section Numbering

Verify that all numbered headings are consecutive with no gaps, at every level.

**How to check:**
1. Extract all headings with `grep -n "^#" <file>`, filtering out code blocks.
2. For top-level numbered sections (e.g., `## 1.`, `## 2.`), verify they run 1, 2, 3, … with no skips.
3. For each subsection group (e.g., `### 2.1`, `### 2.2`), verify they run .1, .2, .3, … with no skips.
4. Verify subsection prefixes match their parent (e.g., all subsections of `## 3.` start with `3.`).

**Common issues:**
- A section was added or removed but siblings were not renumbered.
- A section was moved and the old number was left behind.
- Subsection numbering restarts from the wrong parent.

---

## Check 2 — TOC Completeness

Verify that every numbered heading in the document has a corresponding entry in the Table of Contents, and vice versa — no heading is missing from the TOC, and no TOC entry points to a non-existent heading.

**How to check:**
1. Extract all headings (excluding TOC heading itself and any `<!-- omit in toc -->` headings).
2. Extract all TOC entries.
3. Confirm a 1-to-1 match.

---

## Check 3 — TOC Anchor Links

Verify that every TOC link anchor correctly resolves to its target heading using **GitHub-Flavored Markdown (GFM) anchor rules**:

1. Take the heading text (strip any inline code backticks, keeping the inner text).
2. Convert to lowercase.
3. Remove every character that is not a letter, digit, space, or hyphen.
4. Replace spaces with hyphens.
5. The em-dash `—` is stripped, leaving a double-hyphen gap between its surrounding words.

**Example:**
- `## 2. Phase A — Scaffold a Project` → `#2-phase-a--scaffold-a-project`
- `### 1.1. Install \`agents-cli\`` → `#11-install-agents-cli`

Compare each computed anchor against what the TOC actually contains. Report or fix any mismatch.

---

## Check 4 — HTML Anchor Validity

For documents that use `<a id="..."></a>` anchors (e.g., for cross-references not tied to headings):

1. Collect every `<a id="...">` definition.
2. Collect every `(#...)` reference that targets one of those IDs (i.e., not a heading anchor).
3. Verify every reference has a matching definition, and every definition has at least one reference.

Report orphaned anchors (defined but never linked to) and broken references (linked but not defined).

---

## Check 5 — Spelling (Tech-Aware)

Extract prose text by stripping:
- Fenced code blocks (``` ``` ```)
- Inline code (`` ` `` ... `` ` ``)
- URLs (`https?://...`)
- HTML tags
- Markdown link syntax `[text](url)`
- Markdown image syntax `![alt](url)`

Then scan the remaining prose for spelling errors.

**Do NOT flag as errors:**
- Technical abbreviations: CLI, SDK, API, GCP, ADK, CI/CD, LLM, IDE, GKE, IAM, HTTP, URL, JSON, YAML, etc.
- Tool/product names: Terraform, Kubernetes, Dockerfile, Vertex AI, Cloud Run, Cloud Build, etc.
- Package/command names: `uvicorn`, `fastapi`, `pyproject`, `evalset`, `rubric`, etc. (even when appearing outside backticks in prose)
- File extensions and paths when discussed in prose: `.env`, `.gitignore`, `.json`, etc.
- Domain jargon that is standard in the field: observability, evalset, rubrics, symlink, subcommand, metadata, hot-reload, etc.
- Intentional informal register (e.g., "Gotcha" in a "Key Gotchas" section)

**Do flag:**
- Misspelled common English words (e.g., `explicity` → `explicitly`, `recieve` → `receive`)
- Accidentally merged words (e.g., `Withose` → `With those`)

If `aspell` or `hunspell` is available, run: `cat <file> | aspell list --lang=en_US --mode=markdown | sort -u`
and filter the output against the tech-term allowlist above.
Otherwise, extract and scan prose manually using a Python/sed script.

---

## Check 6 — Basic Prose Grammar

While extracting prose for spelling (Check 5), also scan for common grammatical errors:

- Missing prepositions (e.g., `resulting this` → `resulting in this`)
- Missing articles where clearly required (e.g., `in previous section` → `in the previous section`)
- Subject-verb agreement errors
- Duplicate words (e.g., `the the`)

**Do NOT flag:**
- Intentional terse style common in technical writing (omitting articles is sometimes acceptable)
- Non-native phrasing that is unambiguous and readable

---

## Execution Order

Run checks in this order; fix as you go so later checks see the corrected state:

1. Section Numbering
2. TOC Completeness
3. TOC Anchor Links
4. HTML Anchor Validity
5. Spelling
6. Basic Prose Grammar

After all checks, report a summary table:

| Check | Issues Found | Fixed |
|-------|-------------|-------|
| Section Numbering | N | N |
| TOC Completeness | N | N |
| TOC Anchor Links | N | N |
| HTML Anchor Validity | N | N |
| Spelling | N | N |
| Prose Grammar | N | N |

If any issue required a judgment call and was not auto-fixed, list it explicitly for the user.
