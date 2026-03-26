# WellVerse Architecture (C1-C4)

## C1 -- System Context

WellVerse is a vetted wellness marketplace connecting seekers with guides across six wellness verticals.

```mermaid
graph TB
    Seeker["Seeker<br/>(Person seeking wellness guidance)"]
    Guide["Guide<br/>(Vetted wellness practitioner)"]
    Admin["Admin<br/>(Platform operator)"]

    WV["WellVerse<br/>(Wellness Marketplace Platform)"]

    Stripe["Stripe<br/>(Payment processing)"]
    SendGrid["SendGrid<br/>(Transactional email)"]
    Browser["Browser localStorage<br/>(JWT token storage)"]

    Seeker -->|Browses guides, books sessions,<br/>tracks progress, joins circles| WV
    Guide -->|Manages bookings, receives<br/>messages, builds profile| WV
    Admin -->|Reviews applications, manages<br/>content, views analytics| WV

    WV -->|Creates checkout sessions,<br/>processes payments| Stripe
    WV -->|Sends booking confirmations,<br/>waitlist & application emails| SendGrid
    WV -->|Stores JWT token<br/>for authentication| Browser

    Stripe -->|Webhook events:<br/>payment success, subscription changes| WV
```

### External Actors

| Actor | Role | Interaction |
|-------|------|-------------|
| Seeker | End user seeking wellness guidance | Browse guides, book sessions, message guides, track progress, join circles |
| Guide | Vetted wellness practitioner | Receive bookings, respond to messages, manage availability |
| Admin | Platform operator | Review guide applications, manage library/circles/blog, view analytics |
| Stripe | Payment processor | Checkout sessions, payment intents, subscription management, webhooks |
| SendGrid | Email service | Waitlist confirmations, booking confirmations, application notifications |
| Browser | Client-side storage | JWT token persistence in localStorage |

---

## C2 -- Container Diagram

```mermaid
graph TB
    subgraph "Client"
        SPA["React SPA<br/>(Vite + Tailwind CSS 3.4)<br/>Port 5190"]
    end

    subgraph "Server"
        API["FastAPI Backend<br/>(Python 3.12, Uvicorn)<br/>Port 8026"]
        DB[("SQLite Database<br/>(wellverse.db)<br/>Auto-created on startup")]
        Uploads["Static File Storage<br/>(uploads/ directory)<br/>Avatars & guide photos"]
    end

    subgraph "External Services"
        Stripe["Stripe API"]
        SendGrid["SendGrid API"]
    end

    SPA -->|"REST API calls<br/>Bearer JWT auth<br/>/api/*"| API
    API -->|"SQLAlchemy ORM<br/>sync sessions"| DB
    API -->|"Pillow resize + save<br/>FileResponse serve"| Uploads
    API -->|"stripe SDK<br/>checkout + webhooks"| Stripe
    API -->|"sendgrid SDK<br/>transactional emails"| SendGrid
    SPA -->|"Serves built dist/<br/>in production"| API
```

### Container Details

| Container | Technology | Purpose |
|-----------|-----------|---------|
| React SPA | React 18, Vite 6, Tailwind CSS 3.4, React Router 6 | Single-page application with 18 pages |
| FastAPI Backend | Python 3.12, FastAPI 0.115, Uvicorn 0.34, SQLAlchemy 2.0 | REST API with 17 routers + health endpoint |
| SQLite Database | SQLite via SQLAlchemy | Persistent storage, auto-created and seeded on first run |
| Static File Storage | Local filesystem (`uploads/`) | Avatar images (400x400) and guide photos (600x600) |
| Stripe API | Stripe SDK 11.4 | Payment intents, checkout sessions, subscription management |
| SendGrid API | SendGrid SDK 6.11 | Transactional emails (booking, waitlist, application notifications) |

---

## C3 -- Component Diagram (Backend)

```mermaid
graph TB
    subgraph "FastAPI Application"
        subgraph "Core Routers"
            Auth["Auth Router<br/>/api/auth<br/>register, login, me"]
            Vert["Verticals Router<br/>/api/verticals<br/>list, detail"]
            Guides["Guides Router<br/>/api/guides<br/>list, detail, search"]
            Book["Bookings Router<br/>/api/bookings<br/>create, list, availability, status"]
            Rev["Reviews Router<br/>/api/guides/:id/reviews<br/>create, list"]
            Wait["Waitlist Router<br/>/api/waitlist<br/>join, list"]
            App["Applications Router<br/>/api/applications<br/>submit, list"]
            Stats["Stats Router<br/>/api/stats<br/>platform metrics"]
        end

        subgraph "Feature Routers"
            Msg["Messaging Router<br/>/api/messages<br/>send, conversations, history"]
            Notes["Notes Router<br/>/api/notes<br/>create, list, detail"]
            Prog["Progress Router<br/>/api/progress<br/>create, list, report"]
            Lib["Library Router<br/>/api/library<br/>list, detail, filter"]
            Circ["Circles Router<br/>/api/circles<br/>list, join, leave"]
            Pay["Payments Router<br/>/api/payments<br/>checkout, webhook, history"]
            Upl["Uploads Router<br/>/api/uploads<br/>avatar, guide photo, serve"]
            Adm["Admin Router<br/>/api/admin<br/>dashboard, users, applications,<br/>guide analytics, content management"]
            Blog["Blog Router<br/>/api/blog<br/>list published, get by slug"]
        end

        subgraph "Infrastructure"
            AuthMW["Auth Middleware<br/>JWT decode, get_current_user,<br/>require_user, require_admin"]
            Email["Email Service<br/>SendGrid integration<br/>with logging fallback"]
            Config["Config Module<br/>Environment variables<br/>JWT, Stripe, SendGrid, Uploads"]
            Seed["Seed Module<br/>6 verticals, 14 guides,<br/>6 library items, 6 circles,<br/>3 blog posts, 1 admin user"]
        end

        subgraph "Data Layer"
            ORM["SQLAlchemy ORM<br/>15 models<br/>sync Session"]
            SQLite[("SQLite<br/>wellverse.db")]
        end
    end

    Auth --> AuthMW
    Book --> AuthMW
    Msg --> AuthMW
    Notes --> AuthMW
    Prog --> AuthMW
    Circ --> AuthMW
    Pay --> AuthMW
    Upl --> AuthMW
    Adm --> AuthMW

    Book --> Email
    Wait --> Email
    App --> Email

    AuthMW --> ORM
    ORM --> SQLite
```

### Component Inventory

| Component | Prefix | Endpoints | Auth Level |
|-----------|--------|-----------|------------|
| Auth | `/api/auth` | 4 | None (register/login), User (me) |
| Verticals | `/api/verticals` | 2 | None |
| Guides | `/api/guides` | 2 | None |
| Bookings | `/api/bookings` | 4 | Optional user / None |
| Reviews | `/api/guides/:id/reviews` | 2 | Optional user |
| Waitlist | `/api/waitlist` | 2 | None |
| Applications | `/api/applications` | 2 | None |
| Messaging | `/api/messages` | 3 | User |
| Notes | `/api/notes` | 3 | User |
| Progress | `/api/progress` | 3 | User |
| Library | `/api/library` | 2 | None |
| Circles | `/api/circles` | 3 | None (list), User (join/leave) |
| Payments | `/api/payments` | 3 | User (checkout/history), None (webhook) |
| Uploads | `/api/uploads` | 3 | User (upload), None (serve) |
| Admin | `/api/admin` | 8 | Admin |
| Blog | `/api/blog` | 2 | None |
| Stats | `/api/stats` | 1 | None |
| Health | `/api/health` | 1 | None |

---

## C4 -- Code Level

### Backend Entity Relationships

```mermaid
classDiagram
    class User {
        +int id
        +str email
        +str password_hash
        +str full_name
        +str role [seeker|guide|admin]
        +str tier [free|seeker|committed]
        +str stripe_customer_id
        +str stripe_subscription_id
        +str avatar_url
        +int guide_id FK
        +datetime created_at
    }

    class Vertical {
        +str id PK
        +str label
        +str emoji
        +str color
        +str accent
        +str status [live|early]
        +str eta
        +str tagline
        +text manifesto
        +text why
        +str stat
        +json guide_types
        +json sub_pillars
    }

    class Guide {
        +int id
        +str name
        +str role
        +str emoji
        +str vertical_id FK
        +str color
        +text quote
        +text bio
        +json methods
        +str price
        +float rating
        +int review_count
        +str location
        +bool is_preview
        +bool is_verified
        +str photo_url
        +json availability
        +datetime created_at
    }

    class Booking {
        +int id
        +int guide_id FK
        +int user_id FK
        +str seeker_name
        +str seeker_email
        +str booking_type [intro|session]
        +str date_time
        +int duration_minutes
        +text notes
        +str status [pending|confirmed|completed|cancelled]
        +str payment_intent_id
        +int amount_cents
        +datetime created_at
    }

    class Review {
        +int id
        +int guide_id FK
        +int user_id FK
        +str author
        +float rating
        +text comment
        +datetime created_at
    }

    class WaitlistEntry {
        +int id
        +str email
        +str vertical_id FK
        +datetime created_at
    }

    class GuideApplication {
        +int id
        +str full_name
        +str email
        +str phone
        +str city_country
        +str role_specialty
        +text qualifications
        +str years_practice
        +str website
        +text why_wellverse
        +text approach
        +text first_session
        +str status [pending|approved|declined]
        +text admin_notes
        +int reviewed_by FK
        +datetime reviewed_at
        +datetime created_at
    }

    class Conversation {
        +int id
        +int user_id FK
        +int guide_id FK
        +datetime last_message_at
        +datetime created_at
    }

    class Message {
        +int id
        +int conversation_id FK
        +int sender_id FK
        +int guide_id FK
        +text content
        +bool read
        +datetime created_at
    }

    class SessionNote {
        +int id
        +int booking_id FK
        +int user_id FK
        +text content
        +int mood_before [1-5]
        +int mood_after [1-5]
        +json key_takeaways
        +datetime created_at
    }

    class ProgressEntry {
        +int id
        +int user_id FK
        +str vertical_id FK
        +str entry_type [goal|milestone|reflection|metric]
        +str title
        +text content
        +float value
        +datetime created_at
    }

    class LibraryItem {
        +int id
        +str title
        +text description
        +str content_type [article|video|exercise|meditation]
        +str vertical_id FK
        +text body
        +str media_url
        +int duration_minutes
        +str author
        +json tags
        +bool is_premium
        +datetime created_at
    }

    class Circle {
        +int id
        +str name
        +text description
        +str vertical_id FK
        +int host_guide_id FK
        +int max_members
        +str schedule
        +bool is_premium
        +str image_url
        +datetime created_at
    }

    class CircleMembership {
        +int id
        +int circle_id FK
        +int user_id FK
        +datetime joined_at
    }

    class Payment {
        +int id
        +int user_id FK
        +int booking_id FK
        +str stripe_payment_intent_id
        +int amount_cents
        +str currency
        +str status [pending|succeeded|failed|refunded]
        +str description
        +datetime created_at
    }

    class BlogPost {
        +int id
        +str title
        +str slug UNIQUE
        +text excerpt
        +text body
        +str author
        +str cover_image_url
        +json tags
        +bool published
        +datetime published_at
        +datetime created_at
    }

    User "1" --> "*" Booking : books
    User "1" --> "*" Message : sends
    User "1" --> "*" SessionNote : writes
    User "1" --> "*" ProgressEntry : tracks
    User "1" --> "*" CircleMembership : joins
    User "0..1" --> "0..1" Guide : linked via guide_id

    Vertical "1" --> "*" Guide : contains
    Vertical "1" --> "*" WaitlistEntry : has waitlist
    Vertical "1" --> "*" LibraryItem : categorizes
    Vertical "1" --> "*" Circle : categorizes

    Guide "1" --> "*" Review : receives
    Guide "1" --> "*" Booking : has
    Guide "1" --> "*" Message : receives
    Guide "1" --> "0..1" Circle : hosts

    Booking "1" --> "*" SessionNote : has notes
    Booking "1" --> "0..1" Payment : paid via

    Conversation "1" --> "*" Message : contains
    Circle "1" --> "*" CircleMembership : has members
```

### Frontend Component Tree

```mermaid
graph TB
    App["App"]
    App --> AuthProvider["AuthProvider (Context)"]
    App --> Nav["Nav"]
    App --> Footer["Footer"]
    App --> GuideModal["GuideModal"]
    App --> GuidedTour["GuidedTour"]

    subgraph "Public Routes"
        Home["/ Home"]
        Explore["/explore Explore"]
        VD["/vertical/:id VerticalDetail"]
        Browse["/browse Browse"]
        ForGuides["/for-guides ForGuides"]
        Apply["/apply Apply"]
        SignIn["/signin SignIn"]
        Blog["/blog Blog"]
        BlogPost["/blog/:slug BlogPost"]
        Privacy["/privacy Privacy"]
        Terms["/terms Terms"]
    end

    subgraph "Authenticated Routes"
        Dashboard["/dashboard Dashboard"]
        Messages["/messages Messages"]
        Library["/library Library"]
        Circles["/circles Circles"]
        Settings["/settings Settings"]
    end

    subgraph "Admin Route"
        Admin["/admin Admin"]
    end

    App --> Home
    App --> Explore
    App --> VD
    App --> Browse
    App --> ForGuides
    App --> Apply
    App --> SignIn
    App --> Blog
    App --> BlogPost
    App --> Privacy
    App --> Terms
    App --> Dashboard
    App --> Messages
    App --> Library
    App --> Circles
    App --> Settings
    App --> Admin
```

### Design Token System

The frontend uses a custom color palette defined in `tokens.js`:

| Token | Hex | Usage |
|-------|-----|-------|
| void | `#0A0A08` | Page background |
| ink | `#111410` | Card backgrounds |
| ember | `#1A1A14` | Elevated surfaces |
| parchment | `#F2EDE4` | Primary text |
| dust | `#C4BAA8` | Secondary text |
| amber | `#C8923A` | Accent / CTA |
| gold | `#E2B96F` | Highlights |
| moss | `#3A5A40` | Fitness / Body vertical |
| sage | `#6A9E72` | Fitness accent |
| sky | `#4A7A9A` | Mental Wellness vertical |
| teal | `#2A6A60` | Brand / links |
| wine | `#6A2A40` | Relationships vertical |
| olive | `#5A7A2A` | Nutrition vertical |
| plum | `#5A3A6A` | Beauty vertical |
| indigo | `#2A3A7A` | Passion Circles vertical |

### Tier System

| Tier | Price | Key Limits |
|------|-------|-----------|
| Explorer (free) | Free forever | 1 circle, 1 intro call, browse only |
| Seeker | Pay per session | Unlimited bookings, messaging, notes, progress |
| Committed | $49/month | Unlimited circles, priority booking, 10% off sessions |
