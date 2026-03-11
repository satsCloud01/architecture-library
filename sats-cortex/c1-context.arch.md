---
name: "C1 – System Context"
project: "Sats Cortex"
project_slug: "sats-cortex"
project_url: "https://cortex.satszone.link"
github: ""
category: "productivity"
type: "c4-context"
icon: "🧠"
tags: [FastAPI, React, S3, PyMuPDF]
---

# C1 — System Context

> **Scope:** The entire SatsKnowledge Cortex system and everything it talks to.

```mermaid
C4Context
    title System Context — SatsKnowledge Cortex

    Person(user, "User (Sats)", "Single owner of the knowledge base. Adds, organises, searches, and reads all content.")

    System_Boundary(cortex_boundary, "SatsKnowledge Cortex") {
        System(cortex, "SatsKnowledge Cortex", "Personal knowledge management web application. Stores documents, media, web links, and notes with full-text search and time-limited public sharing.")
    }

    System_Ext(s3, "AWS S3", "Cloud object storage. Holds all uploaded binary files (PDFs, DOCX, video, audio) in production.")
    System_Ext(youtube, "YouTube", "Video hosting platform. Provides embedded iframe players for saved YouTube links.")
    System_Ext(vimeo, "Vimeo", "Video hosting platform. Provides embedded iframe players for saved Vimeo links.")
    System_Ext(soundcloud, "SoundCloud", "Audio hosting platform. Provides embedded iframe players for saved SoundCloud links.")
    System_Ext(web, "External Websites", "Arbitrary web pages saved as link-type articles. Content is fetched at save time for text extraction and summary.")
    System_Ext(recipient, "Link Recipient", "Any person the user shares a public link with. Read-only, unauthenticated, time-limited access.")

    Rel(user, cortex, "Manages knowledge base via browser", "HTTPS")
    Rel(cortex, s3, "Stores and retrieves uploaded files", "AWS SDK / HTTPS")
    Rel(cortex, web, "Fetches page content at save time", "HTTP/HTTPS via httpx")
    Rel(cortex, youtube, "Embeds player for YouTube links", "iframe / HTTPS")
    Rel(cortex, vimeo, "Embeds player for Vimeo links", "iframe / HTTPS")
    Rel(cortex, soundcloud, "Embeds player for SoundCloud links", "iframe / HTTPS")
    Rel(cortex, recipient, "Serves time-limited public read page", "HTTPS")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Key Design Decisions at This Level

| Decision | Choice | Rationale |
|---|---|---|
| Authentication | None | Single-user personal tool; no auth overhead needed |
| Cloud storage | AWS S3 (prod) / local filesystem (dev) | Free-tier S3 sufficient; local filesystem for zero-cost dev |
| Link archiving | Fetch + store text at save time | Full-text search works offline; no crawl quota issues |
| Media embeds | Delegated to hosting platform iframes | No video transcoding or hosting cost |
| Public sharing | Server-rendered HTML page, 30-min expiry token | No login required for recipients; automatic expiry prevents link rot |
