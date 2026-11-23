# HTDC Schema Overview

**Status**: Draft
**Version**: 0.1
**Normative:** Yes

---

This is the overview for the **HTDC Schema** specification, a versioned, JSON-serializable document model for composing documents that must render consistently to multiple output media, principally print (PDF) and web (HTML), from a single source.

HTDC Schema is the contract between document *producers* (markup parsers, editors, CMS exporters, programmatic builders, LLM output adapters) and document *consumers* (renderers, validators, linters, converters).

## 1. Introduction

### 1.1 What HTDC is

HTDC (_HyperText Document Composition_) is an ecosystem of specifications and tools. HTDC Schema is its contract; HTDC Lang is a producer; the HTDC renderers are consumers. No component is privileged: the schema is the product, and every other component depends on it and on nothing else in the ecosystem.

The ecosystem is based on HTDC Schema —the HTDC Doc (metadata + AST) specification— and HTDC Lang — the HTDC markup language. The user writes a simple markup with commands and processing it through the HTDC ecosystem creates a consistent hypertext document from it:

```htdc
/filename my_paper

/title My Paper
/subtitle Made with HTDC

/img ./logo.png { width: 0.25 }

/h1 Why HTDC?
/h2 The acronym

*HyperText Document Composition*. With it I can create consistent hypertext documents for different platforms.
```

Every `/command` binds contents, config and values to a HTDC Doc node. A producer would parse HTDC Lang, emit an HTDC Doc instance, and a consumer would render it as a PDF/HTML document.

### 1.2 Quickstart

```json
{
    "schemaVersion": "0.1",
    "meta": {
        "filename": "my_paper",
        "title": "My Paper",
        "subtitle": "Made with HTDC",
        "authors": ["sadgabi20"],
        "date": "2026-11-23T00:00:00Z",
        "lang": "en",
        "keywords": ["document", "htdc"]
    },
    "root": [{
        "id": "section1",
        "type": "section",
        "contents": [
            {
                "id": "chapter1",
                "type": "h1",
                "contents": [{"type": "text", "value": "Chapter 1" }],
                "config": {
                    "enum": true
                }
            },
            {
                "type": "p",
                "contents": [
                    { "type": "text", "value": "This is the first paragraph of Chapter 1." },
                    { "type": "text", "value": "This part of the paragraph is in italics.", "marks": [{ "type": "italic" }] }
                ]
            }
        ]
    }]
}
```

> `enum: true` means the heading participates in numbering; the actual numbers are computed at render time

Every document carries a required `schemaVersion` in `MAJOR.MINOR` form; consumers reject unknown majors and accept unknown minors of known majors, ignoring unrecognized fields.

### 1.3 Design goals

- **One source, consistent output.** The same HTDC Doc renders consistently to PDF and HTML; structure, not the renderer, is the source of truth.
- **Semantic, not visual.** No fonts, colors, sizes, or page geometry in the document; styling is a render-time profile.
- **Resolve at render, never at author.** Numbering, cross-reference text, TOC entries, and citation strings are computed by consumers, never stored.
- **No escape hatches.** No `rawHtml` or opaque strings; extensions are namespaced and ignorable.

## 2. Features

HTDC Schema is a standard for the structure of hypertext documents with metadata, inspired by HTML's element model and by the structure of academic papers. This specification defines the following categories:

| Category    | What it is                | Examples                           | Specified in   |
|-------------|---------------------------|------------------------------------|----------------|
| Envelope    | Top-level document fields | `schemaVersion`, `meta`, `root`    | `01-envelope`  |
| Meta        | Document metadata         | `title`, `authors`, `date`, `lang` | `01-envelope`  |
| Block node  | Structural content        | `section`, `h1`, `p`, `img`        | `02-blocks`    |
| Inline node | Content within blocks     | `text`, `ref`, `footnoteRef`       | `03-inlines`   |
| Mark        | Inline annotation         | `italic`, `bold`, `link`, `code`   | `04-marks`     |


## 3. Terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in RFC 2119.

**HTDC Doc** — A JSON object conforming to the envelope described in `01-envelope.md`, containing `schemaVersion`, `meta`, and `root`.

**AST** — The tree of nodes rooted at the document's `root` array. Used interchangeably with "document tree."

**Node** — A JSON object within the AST with a required `type` field. Nodes are either **block** nodes (which participate in flow and may contain other blocks or inlines) or **inline** nodes (which occur within a block's inline content).

**Mark** — A non-structural annotation on inline text (bold, italic, link, etc.), expressed as an element of a text node's `marks` array. Marks do not nest structurally; they are a flat set per text node.

**ID** — A string identifier attached to a node, used as a target for references. IDs are document-scoped and MUST be unique within a document.

**Reference** — A node or mark that points to a node by ID, deferring text resolution to render time.

**Producer** — Any process that emits an HTDC Doc.

**Consumer** — Any process that accepts an HTDC Doc and produces output or analysis.

**Renderer** — A consumer that produces a rendered output (HTML, PDF, DOCX, EPUB, etc.).

**Style profile** — A renderer-specific input, separate from the document, that supplies visual configuration: fonts, colors, spacing, page geometry, numbering formats, and so on. Style profiles are explicitly *not* part of this specification.

**Normalization** — The deterministic transformation of a valid document into a canonical form, such that two semantically equivalent documents produce identical normalized ASTs.

**Conformance** — The property of a producer or consumer meeting the requirements of `09-conformance.md`.
