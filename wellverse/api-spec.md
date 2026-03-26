# WellVerse API Specification

Base URL: `/api`

## Authentication

JWT Bearer token passed via `Authorization: Bearer <token>` header. Token stored in browser `localStorage` under key `wellverse_token`.

Auth levels:
- **None**: No authentication required
- **User**: Requires valid JWT (enforced by `require_user`)
- **Admin**: Requires valid JWT with `role=admin` (enforced by `require_admin`)
- **Optional**: Token read if present via `get_current_user`, but endpoint works without it

---

## Health

### GET /health

Returns application health status.

- **Auth**: None
- **Response**: `{ "status": "ok", "app": "WellVerse", "version": "2.0.0" }`

---

## Auth Router (`/api/auth`)

### POST /auth/register

Create a new user account.

- **Auth**: None
- **Request Body**:
  ```json
  { "email": "string", "password": "string", "full_name": "string" }
  ```
- **Response** (201):
  ```json
  {
    "access_token": "string",
    "token_type": "bearer",
    "user": { "id": 1, "email": "...", "full_name": "...", "role": "seeker", "tier": "free", "avatar_url": null, "guide_id": null }
  }
  ```
- **Errors**: 400 (email already registered), 400 (password < 6 chars)
- **Rules**: Default role=`seeker`, tier=`free`

### POST /auth/login

Authenticate and receive a JWT token.

- **Auth**: None
- **Request Body**:
  ```json
  { "email": "string", "password": "string" }
  ```
- **Response** (200): Same as register response
- **Errors**: 401 (invalid email or password)

### GET /auth/me

Get the current authenticated user's profile.

- **Auth**: User
- **Response** (200): `UserOut` object
- **Errors**: 401 (not authenticated)

### PATCH /auth/me

Update the current user's profile.

- **Auth**: User
- **Query Params**: `full_name` (optional), `avatar_url` (optional)
- **Response** (200): Updated `UserOut` object
- **Errors**: 401 (not authenticated)

---

## Verticals Router (`/api/verticals`)

### GET /verticals

List all wellness verticals.

- **Auth**: None
- **Response** (200): Array of `VerticalOut` objects with all fields (id, label, emoji, color, accent, status, eta, tagline, manifesto, why, stat, guide_types, sub_pillars)

### GET /verticals/{vertical_id}

Get a single vertical by ID.

- **Auth**: None
- **Path Params**: `vertical_id` (string, e.g., "mind")
- **Response** (200): `VerticalOut` object
- **Errors**: 404 (vertical not found)

---

## Guides Router (`/api/guides`)

### GET /guides

List guides with optional filters.

- **Auth**: None
- **Query Params**:
  - `vertical_id` (optional string): Filter by vertical
  - `search` (optional string): Search name, role, bio (case-insensitive LIKE)
  - `preview` (optional bool): Filter by preview status
- **Response** (200): Array of `GuideOut` objects

### GET /guides/{guide_id}

Get a single guide by ID.

- **Auth**: None
- **Path Params**: `guide_id` (int)
- **Response** (200): `GuideOut` object (id, name, role, emoji, vertical_id, color, quote, bio, methods, price, rating, review_count, location, is_preview, is_verified, photo_url, availability)
- **Errors**: 404 (guide not found)

---

## Reviews Router (`/api/guides/{guide_id}/reviews`)

### POST /guides/{guide_id}/reviews

Submit a review for a guide.

- **Auth**: Optional
- **Path Params**: `guide_id` (int)
- **Request Body**:
  ```json
  { "author": "string", "rating": 4.5, "comment": "string" }
  ```
- **Response** (201): `ReviewOut` object (id, guide_id, user_id, author, rating, comment, created_at)
- **Errors**: 404 (guide not found), 400 (rating not between 1-5)
- **Side Effects**: Recalculates guide's average rating and review_count from all reviews

### GET /guides/{guide_id}/reviews

List all reviews for a guide.

- **Auth**: None
- **Path Params**: `guide_id` (int)
- **Response** (200): Array of `ReviewOut` objects, ordered by created_at descending

---

## Bookings Router (`/api/bookings`)

### POST /bookings

Create a new booking.

- **Auth**: Optional
- **Request Body**:
  ```json
  {
    "guide_id": 1,
    "seeker_name": "string",
    "seeker_email": "string",
    "booking_type": "intro",
    "date_time": "2025-03-20T10:00:00",
    "notes": "optional string"
  }
  ```
- **Response** (201): `BookingOut` object
- **Errors**: 404 (guide not found), 400 (preview guide), 409 (time slot already booked)
- **Rules**:
  - `intro` bookings: duration=30min, amount=0, auto-confirmed
  - `session` bookings: duration=60min, status=pending
  - Sends booking confirmation email

### GET /bookings

List bookings for the current user.

- **Auth**: Optional (returns user's bookings if authenticated, all if not)
- **Response** (200): Array of `BookingOut` objects, ordered by created_at descending

### GET /bookings/guide/{guide_id}/availability

Get a guide's availability and booked slots.

- **Auth**: None
- **Path Params**: `guide_id` (int)
- **Response** (200):
  ```json
  {
    "guide_id": 1,
    "availability": [{"day": "Monday", "slots": ["09:00", "10:00"]}],
    "booked_slots": ["2025-03-20T10:00:00"]
  }
  ```
- **Errors**: 404 (guide not found)

### PATCH /bookings/{booking_id}/status

Update a booking's status.

- **Auth**: None
- **Path Params**: `booking_id` (int)
- **Query Params**: `status` (string: pending|confirmed|completed|cancelled)
- **Response** (200): `{ "id": 1, "status": "confirmed" }`
- **Errors**: 404 (booking not found), 400 (invalid status value)

---

## Waitlist Router (`/api/waitlist`)

### POST /waitlist

Join the waitlist for a vertical.

- **Auth**: None
- **Request Body**:
  ```json
  { "email": "string", "vertical_id": "nutrition" }
  ```
- **Response** (201): `WaitlistOut` object (id, email, vertical_id, created_at)
- **Rules**: Deduplicates by email + vertical_id (returns existing entry if found). Sends confirmation email on new entry.

### GET /waitlist

List all waitlist entries.

- **Auth**: None
- **Response** (200): Array of `WaitlistOut` objects, ordered by created_at descending

---

## Applications Router (`/api/applications`)

### POST /applications

Submit a guide application.

- **Auth**: None
- **Request Body**:
  ```json
  {
    "full_name": "string",
    "email": "string",
    "role_specialty": "string",
    "phone": "optional",
    "city_country": "optional",
    "qualifications": "optional",
    "years_practice": "optional",
    "website": "optional",
    "why_wellverse": "optional",
    "approach": "optional",
    "first_session": "optional"
  }
  ```
- **Response** (201): `ApplicationOut` object
- **Errors**: 400 (missing required fields: full_name, email, role_specialty)
- **Side Effects**: Sends application received email

### GET /applications

List all applications.

- **Auth**: None
- **Response** (200): Array of `ApplicationOut` objects, ordered by created_at descending

---

## Messaging Router (`/api/messages`)

### POST /messages

Send a message to a guide.

- **Auth**: User
- **Request Body**:
  ```json
  { "guide_id": 1, "content": "Hello!" }
  ```
- **Response** (201): `MessageOut` object (id, conversation_id, sender_id, content, read, created_at)
- **Errors**: 404 (guide not found)
- **Rules**: Auto-creates conversation if none exists for this user-guide pair. Updates conversation's `last_message_at`.

### GET /messages/conversations

List all conversations for the current user.

- **Auth**: User
- **Response** (200): Array of `ConversationOut` objects (id, guide_id, guide_name, guide_emoji, last_message, last_message_at, unread_count), ordered by last_message_at descending

### GET /messages/conversation/{guide_id}

Get all messages in a conversation with a specific guide.

- **Auth**: User
- **Path Params**: `guide_id` (int)
- **Response** (200): Array of `MessageOut` objects, ordered by created_at ascending
- **Side Effects**: Marks all unread messages from the other party as read

---

## Notes Router (`/api/notes`)

### POST /notes

Create a session note.

- **Auth**: User
- **Request Body**:
  ```json
  {
    "booking_id": 1,
    "content": "Session reflection...",
    "mood_before": 3,
    "mood_after": 5,
    "key_takeaways": ["Breathing technique", "Reframing exercise"]
  }
  ```
- **Response** (201): `NoteOut` object
- **Errors**: 404 (booking not found)

### GET /notes

List all notes for the current user.

- **Auth**: User
- **Response** (200): Array of `NoteOut` objects, ordered by created_at descending

### GET /notes/{note_id}

Get a single note.

- **Auth**: User
- **Response** (200): `NoteOut` object
- **Errors**: 404 (note not found or not owned by user)

---

## Progress Router (`/api/progress`)

### POST /progress

Create a progress entry.

- **Auth**: User
- **Request Body**:
  ```json
  {
    "entry_type": "goal",
    "title": "Meditate daily",
    "vertical_id": "mind",
    "content": "10 minutes every morning",
    "value": null
  }
  ```
- **Response** (201): `ProgressOut` object

### GET /progress

List all progress entries for the current user.

- **Auth**: User
- **Response** (200): Array of `ProgressOut` objects, ordered by created_at descending

### GET /progress/report

Get a comprehensive progress report.

- **Auth**: User
- **Response** (200):
  ```json
  {
    "total_bookings": 5,
    "completed_sessions": 3,
    "total_notes": 2,
    "avg_mood_improvement": 1.5,
    "goals": [],
    "milestones": [],
    "recent_entries": []
  }
  ```

---

## Library Router (`/api/library`)

### GET /library

List library items with optional filters.

- **Auth**: None
- **Query Params**:
  - `vertical_id` (optional string)
  - `content_type` (optional string: article|video|exercise|meditation)
  - `search` (optional string): searches title and description
- **Response** (200): Array of `LibraryItemOut` objects, ordered by created_at descending

### GET /library/{item_id}

Get a single library item.

- **Auth**: None
- **Path Params**: `item_id` (int)
- **Response** (200): `LibraryItemOut` object
- **Errors**: 404 (item not found)

---

## Circles Router (`/api/circles`)

### GET /circles

List all circles with membership info.

- **Auth**: Optional (shows `is_member` flag if authenticated)
- **Query Params**: `vertical_id` (optional string)
- **Response** (200): Array of `CircleOut` objects (includes `member_count` and `is_member`)

### POST /circles/{circle_id}/join

Join a community circle.

- **Auth**: User
- **Path Params**: `circle_id` (int)
- **Response** (201): `{ "status": "joined", "circle_id": 1 }`
- **Errors**:
  - 404 (circle not found)
  - 400 (already a member)
  - 400 (circle is full)
  - 403 (premium circle requires paid tier)
  - 403 (free tier allows 1 circle only)

### DELETE /circles/{circle_id}/leave

Leave a community circle.

- **Auth**: User
- **Path Params**: `circle_id` (int)
- **Response** (200): `{ "status": "left", "circle_id": 1 }`
- **Errors**: 404 (not a member)

---

## Payments Router (`/api/payments`)

### POST /payments/create-checkout

Create a Stripe checkout session or simulate payment in dev mode.

- **Auth**: User
- **Request Body**:
  ```json
  { "booking_id": 1, "tier": null }
  ```
  or
  ```json
  { "booking_id": null, "tier": "committed" }
  ```
- **Response** (200):
  - Dev mode: `{ "message": "Dev mode: booking confirmed ..." }`
  - Production (subscription): `{ "checkout_url": "https://checkout.stripe.com/..." }`
  - Production (session payment): `{ "client_secret": "pi_xxx_secret_xxx" }`
- **Errors**: 404 (booking not found), 400 (missing booking_id or tier), 400 (subscription price not configured)
- **Rules**:
  - Without Stripe key: simulates payment, auto-confirms booking, creates Payment record
  - With Stripe key: creates real Stripe CheckoutSession (subscription) or PaymentIntent (booking)
  - Creates Stripe Customer on first use

### POST /payments/webhook

Handle Stripe webhook events.

- **Auth**: None (verified via Stripe signature)
- **Events Handled**:
  - `checkout.session.completed`: upgrades user tier to `committed`, stores subscription ID
  - `payment_intent.succeeded`: confirms booking, creates Payment record
  - `customer.subscription.deleted`: downgrades user tier to `free`
- **Response** (200): `{ "status": "ok" }`
- **Errors**: 400 (invalid webhook signature)

### GET /payments/history

Get the current user's payment history.

- **Auth**: User
- **Response** (200): Array of payment objects (id, amount_cents, currency, status, description, created_at), ordered by created_at descending

---

## Uploads Router (`/api/uploads`)

### POST /uploads/avatar

Upload a user avatar image.

- **Auth**: User
- **Request Body**: Multipart form data with `file` field
- **Response** (200): `{ "url": "/api/uploads/files/avatar_1_abc123.jpg" }`
- **Errors**: 400 (file > 5MB), 400 (invalid format -- only jpg/jpeg/png/webp)
- **Rules**: Image resized to max 400x400px via Pillow. Updates user's `avatar_url`.

### POST /uploads/guide-photo/{guide_id}

Upload a guide profile photo.

- **Auth**: User (must be admin or guide role)
- **Path Params**: `guide_id` (int)
- **Request Body**: Multipart form data with `file` field
- **Response** (200): `{ "url": "/api/uploads/files/guide_1_abc123.jpg" }`
- **Errors**: 403 (not admin/guide), 404 (guide not found), 400 (file > 5MB), 400 (invalid format)
- **Rules**: Image resized to max 600x600px via Pillow. Updates guide's `photo_url`.

### GET /uploads/files/{filename}

Serve an uploaded file.

- **Auth**: None
- **Path Params**: `filename` (string)
- **Response** (200): File content via `FileResponse`
- **Errors**: 404 (file not found)

---

## Blog Router (`/api/blog`)

### GET /blog

List published blog posts.

- **Auth**: None
- **Response** (200): Array of `BlogPostOut` objects (only `published=true`), ordered by published_at descending

### GET /blog/{slug}

Get a single published blog post by slug.

- **Auth**: None
- **Path Params**: `slug` (string)
- **Response** (200): `BlogPostOut` object
- **Errors**: 404 (post not found or not published)

---

## Admin Router (`/api/admin`)

All admin endpoints require admin authentication.

### GET /admin/dashboard

Get platform-wide metrics.

- **Auth**: Admin
- **Response** (200):
  ```json
  {
    "users": 150,
    "guides": 4,
    "pending_applications": 2,
    "total_bookings": 87,
    "completed_sessions": 45,
    "waitlist_entries": 230,
    "total_reviews": 31,
    "total_payments": 12
  }
  ```

### GET /admin/applications

List guide applications with optional status filter.

- **Auth**: Admin
- **Query Params**: `status` (optional: pending|approved|declined)
- **Response** (200): Array of application objects

### PATCH /admin/applications/{app_id}

Approve or decline a guide application.

- **Auth**: Admin
- **Path Params**: `app_id` (int)
- **Request Body**:
  ```json
  {
    "status": "approved",
    "admin_notes": "Strong candidate",
    "vertical_id": "mind",
    "emoji": "...",
    "color": "#4A7A9A",
    "quote": "...",
    "bio": "...",
    "methods": ["Method 1"],
    "price": "$100/session",
    "location": "Virtual"
  }
  ```
- **Response** (200): `{ "id": 1, "status": "approved", "guide_id": 5 }`
- **Errors**: 404 (application not found)
- **Side Effects on approval**: Creates Guide record, creates or upgrades User account (password: `welcome123`)

### GET /admin/users

List all users.

- **Auth**: Admin
- **Response** (200): Array of user objects (id, email, full_name, role, tier, created_at)

### GET /admin/guide-analytics/{guide_id}

Get analytics for a specific guide.

- **Auth**: Admin
- **Path Params**: `guide_id` (int)
- **Response** (200):
  ```json
  {
    "guide_id": 1,
    "name": "Dr. Amara Osei",
    "total_bookings": 15,
    "completed_sessions": 10,
    "cancelled_sessions": 2,
    "completion_rate": 66.7,
    "total_reviews": 5,
    "avg_rating": 4.8,
    "repeat_booking_rate": 0
  }
  ```
- **Errors**: 404 (guide not found)

### POST /admin/library

Create a new library item.

- **Auth**: Admin
- **Request Body**:
  ```json
  {
    "title": "string",
    "description": "string",
    "content_type": "article",
    "vertical_id": "mind",
    "body": "markdown content",
    "duration_minutes": 10,
    "author": "string",
    "tags": ["tag1"],
    "is_premium": false
  }
  ```
- **Response** (201): `{ "id": 1, "title": "..." }`

### POST /admin/circles

Create a new community circle.

- **Auth**: Admin
- **Request Body**:
  ```json
  {
    "name": "string",
    "description": "string",
    "vertical_id": "mind",
    "host_guide_id": 1,
    "max_members": 12,
    "schedule": "Every Monday 7pm GMT",
    "is_premium": false
  }
  ```
- **Response** (201): `{ "id": 1, "name": "..." }`

### POST /admin/blog

Create a new blog post.

- **Auth**: Admin
- **Request Body**:
  ```json
  {
    "title": "string",
    "slug": "url-safe-slug",
    "excerpt": "optional",
    "body": "markdown content",
    "author": "string",
    "cover_image_url": "optional",
    "tags": ["tag1"],
    "published": true
  }
  ```
- **Response** (201): `{ "id": 1, "slug": "url-safe-slug" }`
- **Errors**: 400 (slug already exists)
- **Rules**: If `published=true`, `published_at` is set to current UTC time

### DELETE /admin/guides/{guide_id}

Remove a guide from the platform.

- **Auth**: Admin
- **Path Params**: `guide_id` (int)
- **Response** (200): `{ "status": "deleted" }`
- **Errors**: 404 (guide not found)

---

## Stats Router (`/api/stats`)

### GET /stats

Get public platform statistics.

- **Auth**: None
- **Response** (200):
  ```json
  {
    "founding_members": 847,
    "vetted_guides": 4,
    "intro_sessions": "2,300+",
    "avg_rating": "4.9*",
    "total_verticals": 6,
    "live_verticals": 2
  }
  ```
- **Rules**: Uses real data when available, with minimum seed values (e.g., member count minimum 847)
