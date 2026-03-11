---
name: "C3 – Frontend Components"
project: "Sats Cortex"
project_slug: "sats-cortex"
project_url: "https://cortex.satszone.link"
github: ""
category: "productivity"
type: "c4-component"
icon: "🧠"
tags: [React, Tailwind]
---

# C3 — Frontend Component Diagram

> **Scope:** Internal components of the **Single-Page Application** container.

```mermaid
C4Component
    title Component Diagram — React SPA (Frontend)

    Person(user, "User", "Interacts with the browser UI")
    Container_Ext(api, "FastAPI Backend", "REST/JSON API on /api/v1/*")

    Container_Boundary(spa, "Single-Page Application  [React 18 · TypeScript · Vite · Tailwind · shadcn/ui]") {

        %% ── Layout ───────────────────────────────────────────────────────────
        Component(app, "App.tsx", "React root · QueryClientProvider",
            "Root component. Wraps the whole tree in TanStack React Query client. Renders AppShell.")

        Component(appshell, "AppShell", "layout/AppShell.tsx",
            "Two-column layout. Manages selectedArticleId state. Renders SearchBar in header, LeftPane on the left, RightPane on the right.")

        Component(leftpane, "LeftPane", "layout/LeftPane.tsx",
            "Left sidebar (360px). Hosts Add Article button, FilterBar, and ArticleList. Feeds selected filters and search query down to the list.")

        Component(rightpane, "RightPane", "layout/RightPane.tsx",
            "Right content area (flex). Manages notesOpen state. Renders ArticleMetaBar + ArticleViewer side-by-side with optional NotesPane.")

        %% ── Article List ─────────────────────────────────────────────────────
        Component(articlelist, "ArticleList", "articles/ArticleList.tsx",
            "Renders the scrollable list of articles. Delegates to useArticles or useSearch hook depending on whether a search query is active. Handles bookmark toggle.")

        Component(articlelistitem, "ArticleListItem", "articles/ArticleListItem.tsx",
            "Single row in the list. Shows type icon (PDF, DOCX, HTML, Link, Video, Audio, MD, TXT), title, snippet/summary, tags (max 3), date, bookmark star.")

        %% ── Search / Filter ──────────────────────────────────────────────────
        Component(searchbar, "SearchBar", "search/SearchBar.tsx",
            "Debounced search input (300ms). Updates search query in AppShell state which triggers FTS5 search via useSearch hook.")

        Component(filterbar, "FilterBar", "search/FilterBar.tsx",
            "Type filter chips: All / MD / TXT / PDF / DOCX / HTML / Link / Video / Audio. Bookmark toggle star. Updates filter state in LeftPane.")

        %% ── Article Viewers ──────────────────────────────────────────────────
        Component(viewer, "ArticleViewer", "articles/ArticleViewer.tsx",
            "Dispatcher. Reads article_type and renders the appropriate viewer. Fetches raw text content for text/markdown types via articlesApi.getRaw().")

        Component(v_text, "TextViewer", "viewers/TextViewer.tsx",
            "Renders plain text in a scrollable <pre> block with word-wrap.")

        Component(v_md, "MarkdownViewer", "viewers/MarkdownViewer.tsx",
            "Renders Markdown via react-markdown + remark-gfm + rehype-highlight. Styled prose typography.")

        Component(v_pdf, "PdfViewer", "viewers/PdfViewer.tsx",
            "Paginated PDF viewer using react-pdf (PDF.js). Prev/Next page controls and page counter.")

        Component(v_docx, "DocxViewer", "viewers/DocxViewer.tsx",
            "Fetches raw DOCX bytes, converts to HTML client-side via mammoth.js, injects into a sandboxed div.")

        Component(v_html, "HtmlViewer", "viewers/HtmlViewer.tsx",
            "Fetches HTML bytes, creates a Blob URL (type=text/html, no sandbox), sets as <iframe src>. Scripts execute — full interactivity preserved.")

        Component(v_link, "LinkViewer", "viewers/LinkViewer.tsx",
            "Detects YouTube/Vimeo/SoundCloud URLs → renders embedded iframe player with full controls. All other URLs show a link card with Open Link button.")

        Component(v_video, "VideoViewer", "viewers/VideoViewer.tsx",
            "HTML5 <video controls> element. Full native browser controls: play/pause/seek/volume/mute/fullscreen/playback-rate/PiP.")

        Component(v_audio, "AudioViewer", "viewers/AudioViewer.tsx",
            "HTML5 <audio controls> element with Music icon heading. Full native browser controls: play/pause/seek/volume/mute.")

        %% ── Article Meta + Actions ───────────────────────────────────────────
        Component(metabar, "ArticleMetaBar", "articles/ArticleMetaBar.tsx",
            "Sticky header above the viewer. Inline title editing (click to edit, Enter/Esc). Bookmark toggle, Share button, Download link, Delete (2-click confirm). Tag manager inline.")

        Component(sharebutton, "ShareButton", "articles/ShareButton.tsx",
            "Share icon button. Creates or renews a 30-min public share link via the share API. Popover shows the URL, copy-to-clipboard button, minutes remaining, Renew and Revoke actions.")

        %% ── Notes ────────────────────────────────────────────────────────────
        Component(notespane, "NotesPane", "articles/NotesPane.tsx",
            "Toggleable 288px right panel. Shows note count, Add Note form (collapsible), scrollable NoteCard list. Each NoteCard has inline edit (Cmd+↵ save), hover pencil/trash, 3s delete confirm.")

        %% ── Add Article Modal ────────────────────────────────────────────────
        Component(modal, "AddArticleModal", "articles/AddArticleModal.tsx",
            "shadcn Dialog. Three tabs: Write (text/markdown textarea), Upload (react-dropzone for PDF/DOCX/HTML/MD/TXT/video/audio up to 500MB), Link (URL input with async fetch).")

        Component(tab_write, "WriteTab", "tabs/WriteTab.tsx",
            "Markdown / plain text composer. Type toggle, title field, tag picker, submit.")

        Component(tab_upload, "UploadTab", "tabs/UploadTab.tsx",
            "react-dropzone. Accepts PDF, DOCX, HTML, MD, TXT, MP4, WebM, MOV, AVI, MKV, MP3, WAV, OGG, M4A, FLAC, AAC. Auto-fills title from filename. 500 MB limit.")

        Component(tab_link, "LinkTab", "tabs/LinkTab.tsx",
            "URL input, optional title override, tag picker. Calls articlesApi.createLink() which triggers server-side content fetch.")

        %% ── Tags ─────────────────────────────────────────────────────────────
        Component(tagmanager, "TagManager", "tags/TagManager.tsx",
            "Inline tag picker. Shows existing tags as coloured badges, add new tag inline (name + colour picker). Used in MetaBar and Add modal tabs.")

        Component(tagbadge, "TagBadge", "tags/TagBadge.tsx",
            "Small pill badge rendered with the tag's hex colour. Used in ArticleListItem and TagManager.")

        %% ── API Layer ────────────────────────────────────────────────────────
        Component(api_articles, "articlesApi", "api/articles.ts",
            "list, get, getRaw, createWrite, createLink, upload, update, delete, toggleBookmark, getDownloadUrl, getStats. All calls to /api/v1/articles/*.")

        Component(api_notes, "notesApi", "api/notes.ts",
            "list, create, update, delete for /api/v1/articles/{id}/notes/*.")

        Component(api_shares, "sharesApi", "api/shares.ts",
            "create, get, revoke for /api/v1/articles/{id}/share.")

        Component(api_search, "searchApi", "api/search.ts",
            "search (FTS5) and keywords (prefix suggestions) for /api/v1/search and /api/v1/keywords.")

        Component(api_tags, "tagsApi", "api/tags.ts",
            "list, create, delete for /api/v1/tags/*.")

        %% ── Hooks ────────────────────────────────────────────────────────────
        Component(h_articles, "useArticles hooks", "hooks/useArticles.ts",
            "useArticles (list query), useArticle (single), useCreateWriteArticle, useUploadArticle, useLinkArticle, useUpdateArticle, useDeleteArticle, useToggleBookmark — all backed by React Query.")

        Component(h_search, "useSearch", "hooks/useSearch.ts",
            "useSearch(query, filters) — FTS5 search query enabled only when query.length >= 2.")

        Component(h_notes, "useNotes hooks", "hooks/useNotes.ts",
            "useNotes, useCreateNote, useUpdateNote, useDeleteNote.")

        Component(h_share, "useShare hooks", "hooks/useShare.ts",
            "useShare (get current share, retry:false), useCreateShare, useRevokeShare.")
    }

    %% Relationships ────────────────────────────────────────────────────────────
    Rel(user, app, "Interacts with", "Browser events")

    Rel(app, appshell, "renders")
    Rel(appshell, searchbar, "renders in header")
    Rel(appshell, leftpane, "renders (passes selectedId, setSelectedId)")
    Rel(appshell, rightpane, "renders (passes selectedId)")

    Rel(leftpane, filterbar, "renders")
    Rel(leftpane, articlelist, "renders (passes filters + search query)")
    Rel(leftpane, modal, "opens AddArticleModal")

    Rel(articlelist, articlelistitem, "renders one per article")
    Rel(articlelist, h_articles, "useArticles (no search)")
    Rel(articlelist, h_search, "useSearch (when query active)")

    Rel(rightpane, metabar, "renders")
    Rel(rightpane, viewer, "renders")
    Rel(rightpane, notespane, "renders when notesOpen")

    Rel(metabar, sharebutton, "renders")
    Rel(metabar, tagmanager, "renders for tag editing")
    Rel(metabar, h_articles, "useUpdateArticle, useDeleteArticle, useToggleBookmark")

    Rel(sharebutton, h_share, "useShare, useCreateShare, useRevokeShare")

    Rel(viewer, v_text, "renders for type=text")
    Rel(viewer, v_md, "renders for type=markdown")
    Rel(viewer, v_pdf, "renders for type=pdf")
    Rel(viewer, v_docx, "renders for type=docx")
    Rel(viewer, v_html, "renders for type=html")
    Rel(viewer, v_link, "renders for type=link")
    Rel(viewer, v_video, "renders for type=video")
    Rel(viewer, v_audio, "renders for type=audio")

    Rel(notespane, h_notes, "useNotes, useCreateNote, useUpdateNote, useDeleteNote")

    Rel(modal, tab_write, "tab")
    Rel(modal, tab_upload, "tab")
    Rel(modal, tab_link, "tab")
    Rel(tab_write, h_articles, "useCreateWriteArticle")
    Rel(tab_upload, h_articles, "useUploadArticle")
    Rel(tab_link, h_articles, "useLinkArticle")

    Rel(tagmanager, api_tags, "tagsApi.list, create, delete")

    Rel(h_articles, api_articles, "calls")
    Rel(h_search, api_search, "calls")
    Rel(h_notes, api_notes, "calls")
    Rel(h_share, api_shares, "calls")

    Rel(api_articles, api, "HTTP", "/api/v1/articles/*")
    Rel(api_notes, api, "HTTP", "/api/v1/articles/{id}/notes/*")
    Rel(api_shares, api, "HTTP", "/api/v1/articles/{id}/share")
    Rel(api_search, api, "HTTP", "/api/v1/search")
    Rel(api_tags, api, "HTTP", "/api/v1/tags/*")

    UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="1")
```

## Component File Map

```
frontend/src/
├── App.tsx                                    ← app
├── types/index.ts                             ← shared TypeScript types
├── lib/utils.ts                               ← cn(), formatDate(), formatBytes()
├── api/
│   ├── client.ts                              ← axios instance (baseURL=/api/v1)
│   ├── articles.ts                            ← api_articles
│   ├── notes.ts                               ← api_notes
│   ├── shares.ts                              ← api_shares
│   ├── search.ts                              ← api_search
│   └── tags.ts                                ← api_tags
├── hooks/
│   ├── useArticles.ts                         ← h_articles
│   ├── useSearch.ts                           ← h_search
│   ├── useNotes.ts                            ← h_notes
│   ├── useShare.ts                            ← h_share
│   └── useDebounce.ts
├── components/
│   ├── layout/
│   │   ├── AppShell.tsx                       ← appshell
│   │   ├── LeftPane.tsx                       ← leftpane
│   │   └── RightPane.tsx                      ← rightpane
│   ├── articles/
│   │   ├── ArticleList.tsx                    ← articlelist
│   │   ├── ArticleListItem.tsx                ← articlelistitem
│   │   ├── ArticleViewer.tsx                  ← viewer
│   │   ├── ArticleMetaBar.tsx                 ← metabar
│   │   ├── AddArticleModal.tsx                ← modal
│   │   ├── NotesPane.tsx                      ← notespane
│   │   ├── ShareButton.tsx                    ← sharebutton
│   │   ├── viewers/
│   │   │   ├── TextViewer.tsx                 ← v_text
│   │   │   ├── MarkdownViewer.tsx             ← v_md
│   │   │   ├── PdfViewer.tsx                  ← v_pdf
│   │   │   ├── DocxViewer.tsx                 ← v_docx
│   │   │   ├── HtmlViewer.tsx                 ← v_html
│   │   │   ├── LinkViewer.tsx                 ← v_link
│   │   │   ├── VideoViewer.tsx                ← v_video
│   │   │   └── AudioViewer.tsx                ← v_audio
│   │   └── tabs/
│   │       ├── WriteTab.tsx                   ← tab_write
│   │       ├── UploadTab.tsx                  ← tab_upload
│   │       └── LinkTab.tsx                    ← tab_link
│   ├── search/
│   │   ├── SearchBar.tsx                      ← searchbar
│   │   └── FilterBar.tsx                      ← filterbar
│   └── tags/
│       ├── TagManager.tsx                     ← tagmanager
│       └── TagBadge.tsx                       ← tagbadge
```
