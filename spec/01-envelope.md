# 01 Envelope

**Status:** Draft
**Version:** 0.1
**Normative:** Yes

---

This document specifies the **envelope** of an HTDC Doc: the three top-level fields that every document carries, the rules for each, and the validation errors a consumer must produce when they are violated.

The envelope is deliberately small. Everything else in the schema is either a node inside `root` (specified in `02-blocks` and `03-inlines`) or a render-time concern (style profiles, specified outside this specification).

## 1. Document shape

An HTDC Doc is a JSON object with exactly three top-level fields:

```json
{
  "schemaVersion": "0.1",
  "meta": { "title": "…" },
  "root": []
}
```

| Field           | Type     | Required | Purpose                                                |
|-----------------|----------|----------|--------------------------------------------------------|
| `schemaVersion` | `string` | Yes      | Version of this specification the document conforms to |
| `meta`          | `object` | Yes      | Document metadata (title, authors, language, …)        |
| `root`          | `array`  | Yes      | The document AST: an array of block nodes              |

All three fields are REQUIRED. A document missing any of them is invalid.

**Unknown top-level fields MUST be ignored**, not rejected. This rule exists so that a future minor version can add an optional envelope field without invalidating documents that use it against an older consumer. A consumer that rejects unknown top-level fields is non-conforming.

Unknown fields inside `meta` and inside individual nodes follow the same rule; they are specified where the respective structure is defined.

## 2. `schemaVersion`

`schemaVersion` identifies the version of this specification the document was produced against. It is REQUIRED, is a string, and has the form `MAJOR.MINOR` — two non-negative integers separated by a single period, with no leading zeros and no patch component. Examples: `"0.1"`, `"1.0"`, `"2.14"`.

Consumers MUST handle versions as follows:

| Document version                                                                        | Consumer behavior                          |
|-----------------------------------------------------------------------------------------|--------------------------------------------|
| `MAJOR` equal to a supported major, `MINOR` ≤ the highest supported minor of that major | Accept                                     |
| `MAJOR` equal to a supported major, `MINOR` > the highest supported minor of that major | Accept, ignore unknown fields; SHOULD warn |
| `MAJOR` not supported by the consumer                                                   | Reject with `UNSUPPORTED_MAJOR`            |

The asymmetry is intentional. Minor versions are additive by policy (see `07-versioning.md`), so a consumer can safely read a newer minor and ignore what it does not understand. Major versions are breaking by definition, so a consumer cannot safely read a newer major.

Producers MUST emit the version of the specification they were built against, not the latest version they are aware of.

## 3. `meta`

`meta` carries document-level metadata. It is REQUIRED and is an object. Only title is required inside it; every other field is OPTIONAL.

| Field      | Type        | Required | Description                                             |
|------------|-------------|----------|---------------------------------------------------------|
| `title`    | `string`    | Yes      | Human-readable document title                           |
| `subtitle` | `string`    | No       | Secondary title, usually rendered beneath `title`       |
| `authors`  | `string[]`  | No       | Author names, in display order                          |
| `date`     | `string`    | No       | ISO 8601 date or date-time (see §3.2)                   |
| `lang`     | `string`    | No       | BCP 47 language tag for the document's primary language |
| `keywords` | `string[]`  | No       | Keywords for indexing and metadata output               |
| `filename` | `string`    | No       | Suggested output filename, without extension            |

**Unknown meta fields MUST be ignored**. As with the envelope, this allows minor versions to add optional metadata without breaking older consumers.

An empty `string` is a valid value for any string field; it is not the same as omitting the field. Producers SHOULD omit fields they have no value for, rather than emitting `""` or `null`. `null` is not a valid value for any `meta` field.

### 3.1 `title`

`title` is REQUIRED and MUST be a non-empty string after normalization (see `08-normalization.md`). It is the document's primary title. Renderers use it for the HTML `<title>` element, the PDF document title in the document information dictionary, and any heading the style profile places at the top of the first page. It is metadata, not a block node: it does not appear in root and is not numbered.

A producer that has no `title` SHOULD fail rather than emit an empty one. The value `"Untitled"` is a renderer fallback, not something a producer should write.

### 3.2 `date`

`date` is OPTIONAL and, when present, MUST be a string in ISO 8601 format. Both date-only and date-time forms are accepted:

- `"2026-11-23"` — a calendar date
- `"2026-11-23T00:00:00Z"` — a UTC date-time
- `"2026-11-23T00:00:00-06:00"` — a date-time with an offset

Unix timestamps, JavaScript `Date` objects serialized as numbers, and locale-formatted strings are NOT valid. This is the one place where a producer's convenience is subordinate to cross-language portability: every consumer in every language can parse ISO 8601.

The semantic of `date` is the document's publication or issue date, not the date it was rendered. Renderers MUST NOT substitute the build date when date is absent; they MUST omit the date rather than invent one.

### 3.3 `authors`

`authors` is OPTIONAL and, when present, MUST be an array of strings. Each string is one author's display name.

In version 0.1, authors are strings only. A future major version MAY allow objects (for affiliations, emails, ORCID identifiers, and so on). Producers that need structured author data before then SHOULD encode it in the name string or wait for the major version that adds it; they MUST NOT invent an ad-hoc object shape inside the array, because consumers will not understand it and validators will reject it.

### 3.4 `lang`

`lang` is OPTIONAL and, when present, MUST be a BCP 47 language tag (`"en"`, `"es-MX"`, `"zh-Hant"`). It describes the document's primary language, not the language of an individual node.

When lang is absent, the document's language is unspecified. Consumers MUST NOT assume a language. HTML renderers SHOULD omit the lang attribute on `<html>` when lang is absent, rather than defaulting it. PDF renderers MAY use the value for the document's language metadata when present, and MUST omit it when absent.

### 3.5 `filename`

`filename` is OPTIONAL and, when present, MUST be a string. It is a hint for the output filename, without extension: the string `"my_paper"`, not `"my_paper.pdf"`.

Renderers MAY use it as the default output filename when the user has not specified one. Renderers MUST NOT treat it as a path, MUST NOT honor directory separators in it, and MUST sanitize it before writing to disk. Producers SHOULD emit a filename that is safe on common filesystems (alphanumerics, hyphens, and underscores).

`filename` is a hint, not a requirement. A document without one is valid, and a renderer that ignores filename is conforming.

## 4. `root`

`root` is REQUIRED and MUST be an array. Each element of the array is a block node, as specified in `02-blocks.md`.

`root` MAY be empty. An empty `root` is a valid document: it has metadata but no content. Consumers MUST accept it and SHOULD warn (`EMPTY_ROOT`), because an empty document is almost always a producer bug rather than an intention. A renderer MUST still produce output for it — a PDF with the title page and no body, an HTML document with the title and an empty `<body>`.

**Any block node type is legal at the top level**. `section` is not required and is not privileged: a document whose `root` is `[{ "type":"h1", … }, { "type": "p", … }]` is as valid as one wrapped in a `section`. The choice is stylistic and belongs to the producer.

Block nodes in `root` MUST NOT be nested arbitrarily deep without structure: a producer SHOULD wrap long sequences of blocks in `section` nodes, because sections are what renderers use for page-level decisions (starting a new page, numbering, running headers). A flat `root` with fifty paragraphs is valid but will render as one undifferentiated flow.

The set of legal block node types at the top level is exactly the set specified in `02-blocks.md`. A `root` containing an inline node, a mark, or an unrecognized `type` is invalid.

## 5. Minimal document

The smallest valid HTDC Doc is:

```json
{
  "schemaVersion": "0.1",
  "meta": { "title": "Untitled" },
  "root": []
}
```

This is valid but not useful: it has no content. It exists as a fixture and as the starting point for producers that build documents incrementally.

A more representative minimal document:

```json
{
  "schemaVersion": "0.1",
  "meta": {
    "title": "A Short Note",
    "lang": "en"
  },
  "root": [
    {
      "type": "p",
      "contents": [
        { "type": "text", "value": "Hello, world." }
      ]
    }
  ]
}
```

## 6. Error codes

The following error codes are produced during envelope validation. They are part of the public API of this specification: validators, renderers, and editors branch on them, so they MUST NOT change without a major version bump. Each error carries a JSON Pointer in `path` identifying the offending location.

| Code                     | Condition                                              |
|--------------------------|--------------------------------------------------------|
| `MISSING_SCHEMA_VERSION` | No `schemaVersion` field                               |
| `INVALID_SCHEMA_VERSION` | `schemaVersion` is not a `MAJOR.MINOR` string          |
| `UNSUPPORTED_MAJOR`      | `schemaVersion` major is not supported by the consumer |
| `MISSING_META`           | No `meta` field, or `meta` is not an object            |
| `MISSING_TITLE`          | `meta.title` is absent, `null`, or not a string        |
| `EMPTY_TITLE`            | `meta.title` normalizes to the empty string            |
| `INVALID_META_FIELD`     | A known `meta` field has the wrong type or format      |
| `MISSING_ROOT`           | No `root` field, or `root` is not an array             |
| `INVALID_ROOT_NODE`      | An element of `root` is not a block node               |
| `EMPTY_ROOT`             | `root` is an empty array (warning, not an error)       |

`EMPTY_ROOT` is the only warning in this list. Every other code is a validation failure: a document that triggers any of them is invalid, and a consumer MUST reject it rather than render it.

## 7. What this document does not specify

- The node types that may appear in `root` → `02-blocks.md`
- Inline content and marks inside block nodes → `03-inlines.md`, `04-marks.md`
- How metadata is used by style profiles → outside this specification
- Migration rules between `schemaVersion` majors → `07-versioning.md`
- Normalization of `meta` strings (trimming, Unicode form) → `08-normalization.md`
- Conformance fixtures and the full error code registry → `09-conformance.md`
