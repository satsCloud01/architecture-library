---
name: "Domain Model"
project: "Sats Cortex"
project_slug: "sats-cortex"
project_url: "https://cortex.satszone.link"
github: ""
category: "productivity"
type: "domain-model"
icon: "🧠"
tags: [SQLAlchemy, Pydantic]
---

# Domain Model

> **Scope:** The core business entities, their attributes, relationships, and invariants — independent of any implementation technology.

---

## Entity Relationship Diagram

```mermaid
erDiagram

    ARTICLE {
        string  id              PK  "UUID"
        string  title               "User-visible title"
        string  article_type        "text | markdown | pdf | docx | html | link | video | audio"
        string  storage_key         "Path in file storage (null for link/write articles)"
        string  source_url          "Original URL (link-type only)"
        string  file_name           "Original upload filename"
        int     file_size           "Bytes"
        string  mime_type           "IANA MIME type"
        text    raw_text            "Plain text for FTS indexing"
        string  content_hash        "SHA-256 of stored bytes"
        string  summary             "First ~500 chars of raw_text"
        boolean is_bookmarked       "User favourite flag"
        datetime created_at
        datetime updated_at
    }

    TAG {
        string   id         PK  "UUID"
        string   name           "Unique display name"
        string   color          "Hex colour e.g. #6366f1"
        datetime created_at
    }

    NOTE {
        string   id         PK  "UUID"
        string   article_id FK
        text     content        "Free-form note text (Markdown supported)"
        datetime created_at
        datetime updated_at
    }

    SHARE {
        string   token      PK  "32-byte URL-safe random token"
        string   article_id FK  "Unique — one active share per article"
        datetime expires_at     "UTC · created_at + share_duration_minutes"
        datetime created_at
    }

    ARTICLE_TAGS {
        string article_id  FK
        string tag_id      FK
    }

    ARTICLE ||--o{ ARTICLE_TAGS  : "labelled with"
    TAG     ||--o{ ARTICLE_TAGS  : "applied to"
    ARTICLE ||--o{ NOTE          : "has notes"
    ARTICLE ||--o| SHARE         : "may be shared via"
```

---

## Entity Descriptions

### Article
The central entity. Represents any piece of knowledge the user stores.

| Field | Notes |
|---|---|
| `article_type` | Determines how the content is stored, parsed, and rendered. `link` articles store only a URL and fetched text; `video`/`audio` articles store the binary file but have empty `raw_text`. |
| `storage_key` | Relative path inside the file storage backend. Null for `link` and `write` (text/markdown) articles that were created inline. |
| `raw_text` | Populated by the appropriate parser. Indexed in the FTS5 virtual table for full-text search. Empty for video/audio (binary-only). |
| `content_hash` | SHA-256 of the stored binary. Used to detect duplicate uploads. |
| `summary` | First ~500 characters of `raw_text`. Used for previews in the list pane without fetching the full content. |
| `is_bookmarked` | Boolean flag toggled per article. Drives the bookmark filter in the left pane. |

### Tag
A user-defined label that can be applied to multiple articles.

| Field | Notes |
|---|---|
| `name` | Must be unique across all tags. |
| `color` | Hex string displayed as a coloured pill badge. Chosen by the user in the TagManager. |

### Note
A free-form annotation scoped to a single article.

| Field | Notes |
|---|---|
| `content` | Markdown is accepted and rendered in the Notes pane. |
| Multiplicity | One article can have many notes; notes belong to exactly one article. Cascade-deleted with the article. |

### Share
A time-limited public access grant for one article.

| Field | Notes |
|---|---|
| `token` | Cryptographically random 32-byte URL-safe string. Forms the public URL `/public/{token}`. |
| `article_id` | UNIQUE constraint — only one active share per article at a time. Creating a new share renews the existing one (new token + new expiry). |
| `expires_at` | Set to `now + SHARE_DURATION_MINUTES` (default 30) at creation/renewal. Checked on every public access; expired shares are deleted on access. |

---

## Business Rules

| Rule | Enforced by |
|---|---|
| One share token per article at a time | UNIQUE constraint on `shares.article_id` |
| Expired share tokens are removed on access | `shares.py` router checks `expires_at < now()` and deletes before responding 410 |
| Tags and Notes are deleted when their Article is deleted | `CASCADE DELETE` on FKs |
| `article_tags` junction is cleaned up when either Article or Tag is deleted | `CASCADE DELETE` on both FKs |
| FTS5 index stays in sync with `articles.raw_text` | Three DDL triggers: `after_article_insert`, `after_article_update`, `after_article_delete` |
| File blobs are deleted from storage when an Article is deleted | `article_service.delete_article()` calls `storage.delete(article.storage_key)` before the DB row is removed |
| Upload size limit | `MAX_UPLOAD_MB` env var (default 500 MB) enforced in the articles router |

---

## Article Type Lifecycle

```mermaid
flowchart TD
    A([User action]) --> B{Entry point}

    B --> |POST /articles/write| W[Write flow]
    B --> |POST /articles/upload| U[Upload flow]
    B --> |POST /articles/link| L[Link flow]

    W --> W1[parse_markdown or parse_text]
    W1 --> W2[storage.upload ← .md or .txt blob]
    W2 --> W3[INSERT article\narticle_type=text or markdown]

    U --> U1{Detect file type\nby extension + MIME}
    U1 --> |pdf| P1[parse_pdf · PyMuPDF]
    U1 --> |docx| P2[parse_docx · python-docx]
    U1 --> |html/htm| P3[parse_html · BeautifulSoup4]
    U1 --> |md| P4[parse_markdown]
    U1 --> |txt| P5[parse_text]
    U1 --> |video exts| P6[_media_parse · empty text]
    U1 --> |audio exts| P7[_media_parse · empty text]
    P1 & P2 & P3 & P4 & P5 & P6 & P7 --> U2[storage.upload ← raw bytes]
    U2 --> U3[INSERT article\narticle_type=detected]

    L --> L1[parse_link · httpx async fetch]
    L1 --> L2[INSERT article\narticle_type=link\nno storage_key]

    W3 & U3 & L2 --> DB[(SQLite\nArticle row\nFTS5 index updated\nvia trigger)]
```
