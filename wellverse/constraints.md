# WellVerse Constraints & Decisions

## Technical Constraints

### Runtime
- **Python 3.12** backend with FastAPI 0.115, Uvicorn 0.34
- **Node 20** for frontend build (Vite 6, React 18)
- **SQLite** database via SQLAlchemy 2.0 (sync sessions, no async)
- No external database dependency -- single-file DB at `wellverse.db`

### Frontend
- React 18 with plain JSX (no TypeScript)
- Tailwind CSS 3.4 for styling
- React Router 6 for client-side routing
- No state management library (React Context + useState only)
- No component library -- all components hand-rolled
- Custom design token system in `tokens.js` (dark palette)

### Dependencies
- `greenlet` required for SQLAlchemy on macOS
- `pillow` for image processing (resize uploads)
- `python-jose[cryptography]` for JWT
- `passlib[bcrypt]` + `bcrypt` for password hashing
- `stripe` SDK 11.4 (optional -- graceful fallback without key)
- `sendgrid` SDK 6.11 (optional -- graceful fallback to logging)

---

## Security Decisions

### Authentication
- **JWT tokens** with HS256 algorithm
- Token expiry: **7 days** (`ACCESS_TOKEN_EXPIRE_MINUTES = 10080`)
- Token stored in browser `localStorage` under key `wellverse_token`
- Secret key: environment variable `WELLVERSE_SECRET_KEY` with a dev fallback default
- No refresh token mechanism -- full re-login required after expiry

### Password Security
- **bcrypt** hashing via passlib
- Minimum password length: **6 characters** (enforced on registration only)
- No password complexity requirements beyond length
- Default password for approved guides: `welcome123` (should be changed on first login)

### Authorization Levels
- **None**: Public endpoints (verticals, guides, library, blog, stats, health)
- **Optional user** (`get_current_user`): Bookings, reviews -- works with or without auth
- **Required user** (`require_user`): Messaging, notes, progress, circles, payments, uploads, settings
- **Required admin** (`require_admin`): All `/api/admin/*` endpoints

### API Key Storage
- No server-side API key storage for third-party services beyond environment variables
- Stripe and SendGrid keys are environment variables, not user-provided
- No BYOK pattern -- this is a platform-managed app

### CORS
- **Wide open**: `allow_origins=["*"]`, all methods, all headers, credentials allowed
- Acceptable for dev and single-origin deployment behind a reverse proxy

---

## Payment Strategy

### Stripe Integration
- **Three Stripe event types** handled via webhook:
  1. `checkout.session.completed` -- subscription activation
  2. `payment_intent.succeeded` -- session payment confirmation
  3. `customer.subscription.deleted` -- tier downgrade
- Stripe Customer created on first checkout (stored as `stripe_customer_id`)
- Subscription ID tracked for tier management

### Dev Mode Fallback
- When `STRIPE_SECRET_KEY` is empty (no Stripe configured):
  - Booking payments are auto-confirmed with `amount_cents=0`
  - Tier upgrades are applied immediately without real payment
  - Payment records are created with status `succeeded` and description "Dev mode - free"
  - Webhook endpoint returns `{ "status": "skipped" }`

### Tier Model
| Tier | Price | Implementation |
|------|-------|---------------|
| Explorer (free) | $0 | Default on registration |
| Seeker | Pay per session | No subscription; payment per booking |
| Committed | $49/month | Stripe subscription via `STRIPE_PRICE_COMMITTED` |

---

## Email Strategy

### SendGrid Integration
- Transactional emails for 4 workflows:
  1. **Waitlist confirmation** -- on waitlist join
  2. **Booking confirmation** -- on booking creation
  3. **Application received** -- on guide application submission
  4. **Application decision** -- on admin approval/decline (different templates for each)

### Fallback Behavior
- When `SENDGRID_API_KEY` is empty:
  - All emails are logged to stdout via Python `logging` module
  - Log prefix: `[EMAIL-DEV]` with recipient, subject, and body preview (first 200 chars)
  - `send_email()` returns `True` (success) in fallback mode
- No email queue or retry mechanism

---

## Deployment

### Container Strategy
- **Single Docker container** serving both frontend and backend
- Multi-stage build: Node 20 builds frontend, Python 3.12 slim serves everything
- Frontend `dist/` is mounted as static files by FastAPI in production
- Port: **8026**

### Production Environment
- Deployed on GCP VM via Docker
- Served behind Nginx reverse proxy with SSL
- DNS via AWS Route 53

---

## Data Constraints

### SQLite Limitations
- No concurrent write support -- single-writer model
- No migrations framework -- schema created via `Base.metadata.create_all()` on every startup
- Adding new columns requires manual DB deletion and re-seed, or manual SQLite ALTER TABLE
- No connection pooling tuning

### Seed Data
- Auto-seeded on first run (when `verticals` table is empty):
  - 6 verticals (2 live: mind, body; 4 early: nutrition, relationships, beauty, groups)
  - 14 guides (4 live, 10 preview)
  - 1 admin user (admin@wellverse.app / admin123)
  - 6 library items
  - 6 community circles
  - 3 blog posts
- Seed is idempotent -- skipped if verticals already exist

---

## Rate Limiting

- **None currently implemented**
- No request throttling on any endpoint
- No brute-force protection on login
- No CAPTCHA on public submission forms (waitlist, applications, bookings)

---

## File Upload Constraints

| Constraint | Value |
|-----------|-------|
| Max file size | 5 MB |
| Allowed formats | jpg, jpeg, png, webp |
| Avatar resize | 400x400 px max (thumbnail) |
| Guide photo resize | 600x600 px max (thumbnail) |
| Storage location | `uploads/` directory (local filesystem) |
| Naming | `avatar_{userId}_{uuid8}.{ext}` or `guide_{guideId}_{uuid8}.{ext}` |
| Serving | Via `FileResponse` at `/api/uploads/files/{filename}` |

---

## Circle Constraints

| Rule | Value |
|------|-------|
| Free tier circle limit | 1 circle maximum |
| Committed tier circle limit | Unlimited |
| Premium circle access | Requires `seeker` or `committed` tier |
| Max members per circle | Configurable per circle (default 12) |
| Duplicate membership | Prevented (HTTP 400) |

---

## Booking Rules

| Rule | Implementation |
|------|---------------|
| Double-booking prevention | Cannot book a slot with existing `pending` or `confirmed` booking for same guide |
| Intro auto-confirm | `intro` bookings immediately set to `confirmed` status |
| Preview guide blocking | Cannot book guides with `is_preview=true` (HTTP 400) |
| Intro duration | Fixed at 30 minutes |
| Session duration | Fixed at 60 minutes |
| Intro pricing | Always free (`amount_cents=0`) |
| Auth requirement | Optional -- unauthenticated bookings allowed |

---

## Review Constraints

| Rule | Implementation |
|------|---------------|
| Rating range | 1.0 to 5.0 (server-validated, HTTP 400 on violation) |
| Rating recalculation | Mean of all review ratings recalculated on every new review |
| Review count | Recounted on every new review |
| Duplicate reviews | Not prevented -- same user can review a guide multiple times |
| Auth requirement | Optional -- anonymous reviews allowed (with author name) |

---

## Guide Creation

- Guides are **only** created through admin approval of a GuideApplication
- No self-registration as a guide
- On approval, the admin provides: vertical_id, emoji, color, quote, bio, methods, price, location
- System auto-creates a User account with role=`guide` and password=`welcome123`
- If the applicant email already has a User account, that account is upgraded to role=`guide`
- Guide deletion is admin-only (cascade behavior depends on SQLite FK config)

---

## Tour System

- Guided tour component (`GuidedTour`) auto-shows on first visit after 1.2-second delay
- Tour completion state stored in `localStorage` (key from `LS_KEY` constant)
- Tour can be re-triggered from the Nav component
- Tour steps are auth-aware -- skips authenticated-only steps for unauthenticated users
