---
name: "C4 – Code/Class View"
project: "Sats Cortex"
project_slug: "sats-cortex"
project_url: "https://cortex.satszone.link"
github: ""
category: "productivity"
type: "c4-code"
icon: "🧠"
tags: [SQLAlchemy, Pydantic]
---

# C4 — Code Diagram

> **Scope:** Key classes and functions within the most critical backend components.
> Focus areas: ORM models, storage abstraction, parser contract, service layer, and schema layer.

---

## 1. SQLAlchemy ORM Models (`models.py`)

```mermaid
classDiagram
    class Base {
        <<SQLAlchemy DeclarativeBase>>
    }

    class Article {
        +String id  PK · UUID default
        +String title
        +String article_type  text|markdown|pdf|docx|html|link|video|audio
        +String storage_key   nullable
        +String source_url    nullable
        +String file_name     nullable
        +Integer file_size    nullable
        +String mime_type     nullable
        +Text raw_text        nullable
        +String content_hash  SHA-256 hex
        +String summary       first 500 chars
        +Boolean is_bookmarked  default False
        +DateTime created_at  server_default=now
        +DateTime updated_at  onupdate=now
        +List~Tag~ tags       many-to-many via article_tags
        +List~Note~ notes     one-to-many
        +Share share          one-to-one nullable
    }

    class Tag {
        +String id    PK · UUID default
        +String name  UNIQUE · NOT NULL
        +String color hex string e.g. #6366f1
        +DateTime created_at
        +List~Article~ articles  back-ref
    }

    class article_tags {
        <<junction table>>
        +String article_id  FK → articles.id CASCADE DELETE
        +String tag_id      FK → tags.id CASCADE DELETE
    }

    class Note {
        +String id         PK · UUID default
        +String article_id FK → articles.id CASCADE DELETE
        +Text content      NOT NULL
        +DateTime created_at
        +DateTime updated_at
        +Article article   back-ref
    }

    class Share {
        +String token      PK · secrets.token_urlsafe(32)
        +String article_id FK → articles.id CASCADE DELETE · UNIQUE
        +DateTime expires_at  UTC
        +DateTime created_at  server_default=now
        +Article article   back-ref
    }

    Base <|-- Article
    Base <|-- Tag
    Base <|-- Note
    Base <|-- Share

    Article "many" -- "many" Tag : article_tags
    Article "1" -- "0..*" Note : notes (cascade delete)
    Article "1" -- "0..1" Share : share (cascade delete)
```

---

## 2. Storage Abstraction (`storage/`)

```mermaid
classDiagram
    class StorageBackend {
        <<Protocol>>
        +upload(key: str, data: bytes, content_type: str) Awaitable~None~
        +download(key: str) Awaitable~bytes~
        +delete(key: str) Awaitable~None~
        +get_url(key: str) str
    }

    class LocalStorageBackend {
        -data_dir: Path
        +__init__(data_dir: str)
        +upload(key, data, content_type) async None
        +download(key) async bytes
        +delete(key) async None
        +get_url(key) str  /api/files/{key}/download
    }

    class S3StorageBackend {
        -client: boto3.client
        -bucket: str
        +__init__(bucket, region, access_key, secret_key)
        +upload(key, data, content_type) async None
        +download(key) async bytes
        +delete(key) async None
        +get_url(key) str  presigned S3 GET URL
    }

    StorageBackend <|.. LocalStorageBackend : implements
    StorageBackend <|.. S3StorageBackend : implements
```

---

## 3. Parser Contract (`parsers/`)

```mermaid
classDiagram
    class ParseResult {
        <<dataclass>>
        +str plain_text
        +str summary
        +str title_hint
        +str mime_type
        +int word_count
    }

    class PdfParser {
        +parse_pdf(data: bytes, filename: str) ParseResult
        note "PyMuPDF · iterates all pages · extracts text blocks"
    }

    class DocxParser {
        +parse_docx(data: bytes, filename: str) ParseResult
        note "python-docx · iterates paragraphs · first para as title_hint"
    }

    class HtmlParser {
        +parse_html(data: bytes, filename: str) ParseResult
        note "BeautifulSoup4+lxml · strips scripts/styles · extracts title tag"
    }

    class MarkdownParser {
        +parse_markdown(data: bytes, filename: str) ParseResult
        +parse_text(data: bytes, filename: str) ParseResult
        note "regex H1 as title_hint · full raw text preserved for FTS"
    }

    class LinkParser {
        +parse_link(url: str) Awaitable~ParseResult~
        note "httpx async fetch · browser UA · BeautifulSoup4 text extraction"
    }

    PdfParser ..> ParseResult : returns
    DocxParser ..> ParseResult : returns
    HtmlParser ..> ParseResult : returns
    MarkdownParser ..> ParseResult : returns
    LinkParser ..> ParseResult : returns
```

---

## 4. Pydantic Schemas (`schemas.py`)

```mermaid
classDiagram
    class TagOut {
        +str id
        +str name
        +str color
    }

    class ArticleOut {
        +str id
        +str title
        +str article_type
        +str storage_key
        +str source_url
        +str file_name
        +int file_size
        +str mime_type
        +str summary
        +bool is_bookmarked
        +datetime created_at
        +datetime updated_at
        +List~TagOut~ tags
        +str download_url  computed validator
    }

    class ArticleCreate {
        +str title
        +Literal article_type  text|markdown
        +str content
        +List~str~ tag_ids
    }

    class ArticleLinkCreate {
        +HttpUrl url
        +str title_override  optional
        +List~str~ tag_ids
    }

    class ArticleUpdate {
        +str title          optional
        +bool is_bookmarked optional
        +List~str~ tag_ids  optional
        +str content        optional  text/markdown only
    }

    class ArticleListResponse {
        +List~ArticleOut~ articles
        +int total
    }

    class NoteOut {
        +str id
        +str article_id
        +str content
        +datetime created_at
        +datetime updated_at
    }

    class NoteCreate {
        +str content  min_length=1
    }

    class NoteUpdate {
        +str content  min_length=1
    }

    class ShareOut {
        +str token
        +str article_id
        +datetime expires_at
        +datetime created_at
        +str public_url  computed: {base_url}/public/{token}
        +int minutes_remaining
    }

    ArticleOut "1" o-- "many" TagOut
    ArticleListResponse "1" o-- "many" ArticleOut
```

---

## 5. article_service Key Function Signatures

```mermaid
classDiagram
    class article_service {
        <<module: services/article_service.py>>
        +create_write_article(db, storage, payload: ArticleCreate) Awaitable~Article~
        +create_upload_article(db, storage, file: UploadFile, title, tag_ids) Awaitable~Article~
        +create_link_article(db, storage, payload: ArticleLinkCreate) Awaitable~Article~
        +update_article(db, storage, article: Article, payload: ArticleUpdate) Awaitable~Article~
        +delete_article(db, storage, article: Article) Awaitable~None~
        -_sha256(data: bytes) str
        -_safe_name(filename: str) str
        -_media_parse(filename: str, mime: str) ParseResult
        -_resolve_tags(db, tag_ids: List~str~) Awaitable~List~Tag~~
    }

    class search_service {
        <<module: services/search_service.py>>
        +search_articles(db, query: str, article_type, bookmarked, limit, offset) Awaitable~SearchResponse~
        +get_keywords(db, prefix: str, limit: int) Awaitable~List~str~~
    }
```
