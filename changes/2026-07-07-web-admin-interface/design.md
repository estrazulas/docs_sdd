## Context

HeadroomGate currently exposes user management, team management, API key lifecycle, and usage monitoring exclusively through CLI commands (`headroom auth *`, `headroom usage *`). The proxy serves a read-only dashboard at `/dashboard` (single HTML file with Tailwind CSS + Alpine.js + inline SVG). The auth plugin (`headroom-auth`) provides ASGI middleware that validates `hr_*` API keys, resolves user identity, enforces RBAC, and injects provider keys.

This design adds a `/manage` web interface that reuses the existing `Neo4jAuthStore`, `AuditStore`, and RBAC model while introducing session-based authentication for browser security.

### Constraints

- Must not modify upstream dashboard behavior (keep `/dashboard` unchanged)
- Must reuse existing `Neo4jAuthStore` and `AuditStore` as-is
- No new Python dependencies beyond what the project already provides
- Visual consistency with the existing dashboard (Tailwind CSS, dark/light mode)

## Goals / Non-Goals

**Goals:**
- Session-based admin login using existing `hr_*` API keys (validated via `whoami`)
- httpOnly cookie tokens — API keys never stored in browser
- Multi-page admin interface: users, teams, API keys, usage monitoring
- RBAC-aware: team leads see only their team; admins see everything
- Usage dashboard with top-consumer charts, session drill-down, and full-text search
- Copy-to-clipboard API key distribution format

**Non-Goals:**
- Modifying the existing `/dashboard` HTML or routes
- New database or storage beyond Neo4j + Qdrant
- OAuth, SSO, or password-based authentication
- Mobile app or PWA
- Email notifications or invitation flows
- Real-time WebSocket updates (poll-based is sufficient)

## Decisions

### 1. Architecture: FastAPI router + Jinja2 templates

**Choice**: A dedicated FastAPI router (`headroom/admin/`) with server-rendered Jinja2 templates, mounted at `/manage`.

**Rationale**: Keeps the admin interface within the same FastAPI process — no separate server, no CORS issues, no additional deployment steps. The existing proxy already uses FastAPI (`headroom/proxy/server.py`).

**Alternatives considered**:
- *Separate SPA (React/Vue)*: Overengineering for CRUD forms and charts. Adds npm build step, CORS complexity, and a separate deployment.
- *Single HTML like `/dashboard`*: Too unwieldy for multi-page CRUD with session management. The dashboard pattern works for read-only metrics but not for forms, validation, and multi-entity management.

### 2. Session management: Server-side dict with TTL

**Choice**: In-memory session store (`dict[SessionId, SessionData]`) with background cleanup. Session tokens are random 256-bit hex strings stored as httpOnly, SameSite=Strict cookies.

**Rationale**: No dependency on Neo4j for session storage — sessions are ephemeral and don't need persistence across restarts. In-memory with TTL is the simplest secure approach.

**Session flow**:
1. `POST /manage/login` → validate `hr_*` key via `Neo4jAuthStore.resolve_key_identity()` → create session → `Set-Cookie: headroom_session=<token>; HttpOnly; SameSite=Strict; Path=/manage`
2. Every subsequent request: middleware extracts cookie, looks up session, sets auth contextvars
3. `POST /manage/logout` → delete session → clear cookie

**Alternatives considered**:
- *JWT tokens*: Adds complexity without benefit — sessions are server-side only, no distributed verification needed.
- *Neo4j session nodes*: Adds latency to every page load. Sessions are transient.

### 3. Frontend: Jinja2 + Tailwind CSS CDN + Alpine.js + HTMX

**Choice**: Server-rendered Jinja2 templates with a shared `base.html` layout. Tailwind CSS loaded from CDN (same as dashboard). Alpine.js for dropdowns, modals, toasts. HTMX for form submissions and inline updates without full page reloads.

**Rationale**: Matches the existing dashboard's tech choices (Tailwind, Alpine.js). No build step. Jinja2 is FastAPI's default and requires no additional dependency. HTMX eliminates the need for custom fetch() logic in every form.

**Template structure**:
```
headroom/admin/templates/
  base.html       — layout (sidebar nav, header with user info, dark mode toggle)
  login.html      — standalone login form (no sidebar)
  users.html      — user list + create form
  teams.html      — team list + create form  
  keys.html       — key list + create form + copy-to-clipboard
  usage.html      — usage dashboard (charts, search, session drill-down)
  partials/       — HTMX response fragments (table rows, modals)
```

**Alternatives considered**:
- *Alpine.js SPA with client-side routing*: Single HTML with multiple tabs. Rejected because admin forms are more complex than dashboard metrics — server-rendered templates handle validation errors, flash messages, and redirects more naturally.

### 4. Usage charts: Inline SVG (same pattern as dashboard)

**Choice**: Inline SVG elements generated server-side, rendered with Alpine.js data binding. No charting library.

**Rationale**: The existing dashboard already renders sparklines and trend charts via inline SVG `<path>` elements. Keeping the same approach ensures visual consistency and avoids adding Chart.js or D3 as dependencies.

### 5. CSRF protection: SameSite + Origin check

**Choice**: Rely on `SameSite=Strict` cookies plus an `Origin`/`Referer` header check on mutation endpoints (POST, PUT, DELETE).

**Rationale**: `SameSite=Strict` prevents cross-site request forgery at the browser level. The Origin check provides defense-in-depth. No CSRF tokens needed — simpler UX.

## Risks / Trade-offs

- **In-memory sessions lost on proxy restart** → Acceptable: users re-login with their API key. Sessions are short-lived (12h TTL).
- **No audit trail for admin actions** → Mitigation: log admin actions (create user, revoke key) to the existing `AuditStore` so they appear in usage history.
- **Tailwind CDN dependency** → Mitigation: Admin interface only — if CDN is down, the page looks broken but the proxy keeps running. Acceptable for internal tooling.
- **Session store grows unbounded under high load** → Mitigation: Background task cleans expired sessions every 5 minutes.
