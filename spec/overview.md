# HTDC Schema Overview

**Status**: Draft
**Version**: 0.1

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
