## 1. Module structure and session infrastructure

- [x] 1.1 Create `headroom/admin/` package with `__init__.py`, `router.py`, `session.py`
- [x] 1.2 Implement in-memory session store with TTL and background cleanup (`AdminSessionStore`)
- [x] 1.3 Create `headroom/admin/templates/` directory and `base.html` layout (sidebar nav, header, dark mode toggle)

## 2. Authentication

- [x] 2.1 Implement `POST /manage/login` endpoint — validate `hr_*` key via `Neo4jAuthStore`, create session, set httpOnly cookie
- [x] 2.2 Implement session middleware for `/manage` routes — extract cookie, resolve session, set auth contextvars
- [x] 2.3 Implement `POST /manage/logout` — delete session, clear cookie
- [x] 2.4 Create `login.html` template (standalone form, no sidebar)
- [x] 2.5 Add RBAC guard — redirect non-admin/non-team_lead roles to access denied page

## 3. User management

- [x] 3.1 Implement `GET /manage/api/users` — list users with RBAC scoping (team_lead sees only team)
- [x] 3.2 Implement `POST /manage/api/users` — create user form handler with validation
- [x] 3.3 Implement `PUT /manage/api/users/<id>/status` — deactivate/reactivate user
- [x] 3.4 Create `users.html` template — user list table, create user modal/form, status toggle buttons
- [x] 3.5 Wire HTMX responses for inline form submission and status updates

## 4. Team management

- [x] 4.1 Implement `GET /manage/api/teams` — list teams with member counts
- [x] 4.2 Implement `POST /manage/api/teams` — create team (admin only)
- [x] 4.3 Implement `POST /manage/api/teams/<name>/users` — add user to team
- [x] 4.4 Create `teams.html` template — team list, create form, add-user dropdown

## 5. API key management

- [x] 5.1 Implement `GET /manage/api/keys` — list keys with status, prefix, associated user
- [x] 5.2 Implement `POST /manage/api/keys` — generate new key with TTL, return full key once
- [x] 5.3 Implement `DELETE /manage/api/keys/<id>` — revoke key with confirmation
- [x] 5.4 Create `keys.html` template — key list with status badges, create form, copy-to-clipboard JSON output

## 6. Usage monitoring

- [x] 6.1 Implement `GET /manage/api/usage/summary` — aggregate metrics (total tokens, requests, active users, saved)
- [x] 6.2 Implement `GET /manage/api/usage/top` — top consumers by team or user with time window filter
- [x] 6.3 Implement `GET /manage/api/usage/users/<id>/sessions` — paginated user request history
- [x] 6.4 Implement `GET /manage/api/usage/search` — semantic search via Qdrant with RBAC scoping
- [x] 6.5 Create `usage.html` template — summary cards, SVG bar chart (top consumers), user drill-down table, search input with filters
- [x] 6.6 Implement time window selector (24h / 7d / 30d) with Alpine.js reactive chart updates

## 7. Proxy integration

- [x] 7.1 Register admin router in `headroom/proxy/server.py` at `/manage` path
- [x] 7.2 Add login icon/link to existing `/dashboard` HTML (minimal change — single icon + link)
- [x] 7.3 Add `/manage` to auth middleware bypass for the login page, enforce session on all other `/manage` routes
- [x] 7.4 Wire `AuditStore` for admin action logging (create user, revoke key events)

## 8. Security testing (RBAC bypass & session attacks)

- [x] 8.1 Team lead cannot create user in another team (POST with different team field → 403)
- [x] 8.2 Team lead cannot see users from other teams (GET users returns only own team)
- [x] 8.3 Team lead cannot create teams (POST teams → 403 or button hidden)
- [x] 8.4 Team lead cannot see/manage keys from users in other teams
- [x] 8.5 Developer role cannot access any `/manage` page → redirected to access denied
- [x] 8.6 Viewer role cannot access any `/manage` page → redirected to access denied
- [x] 8.7 Forged session cookie (random token) → redirected to login, no data leaked
- [x] 8.8 Expired session reused → redirected to login
- [x] 8.9 Direct API call without session cookie → 401, no data leaked
- [x] 8.10 Cross-team URL manipulation: team_lead changes user ID in API URL to another team's user → 403 or 404
- [x] 8.11 Cross-team key manipulation: team_lead tries to revoke key from another team's user → 403
- [x] 8.12 Usage search cross-team: team_lead search returns only own team's results, not other teams'
- [x] 8.13 Logout session invalidation: old cookie rejected after logout, cannot access protected routes

## 9. Testing (functional correctness)

- [x] 9.1 Unit tests for `AdminSessionStore` (create, resolve, expire, cleanup)
- [x] 9.2 Integration tests for login/logout flow with mock Neo4j
- [x] 9.3 Integration tests for CRUD API endpoints with RBAC enforcement
- [x] 9.4 Integration tests for usage API endpoints with scope enforcement
- [x] 9.5 Playwright E2E test: admin login → create team → create user → generate key → revoke key
- [x] 9.6 Playwright E2E test: team lead login → scoped views (own team/users only)
- [x] 9.7 Playwright E2E test: usage dashboard loads, chart renders, search works
