## ADDED Requirements

### Requirement: Session creation from API key

The system SHALL accept an `hr_*` API key submitted via login form, validate it against the existing `Neo4jAuthStore.resolve_key_identity()`, and upon successful validation, create a server-side session with a 12-hour TTL.

#### Scenario: Valid API key login

- **WHEN** a user submits a valid `hr_*` API key via `POST /manage/login`
- **THEN** the system SHALL resolve the key identity, create a session, and respond with a `Set-Cookie` header containing an httpOnly, SameSite=Strict session token scoped to `/manage`

#### Scenario: Invalid API key login

- **WHEN** a user submits an invalid, revoked, or expired API key via `POST /manage/login`
- **THEN** the system SHALL return the login page with a generic error message ("Invalid API key. Make sure the key is active and not expired.") and SHALL NOT create a session

### Requirement: Session authentication via FastAPI dependencies

The system SHALL protect all routes under `/manage` (except `/manage/login` and static pages) using FastAPI dependency guards that extract the session token from the cookie header, look up the corresponding session, and raise HTTP 401/403 when the user is missing or has insufficient role.

#### Scenario: Authenticated request

- **WHEN** a request arrives with a valid session cookie
- **THEN** the dependency guard SHALL resolve the session and forward the user data to the handler

#### Scenario: Missing or expired session

- **WHEN** a request arrives without a session cookie, or with an expired/invalid token
- **THEN** the dependency guard SHALL raise `HTTPException(status_code=401)`

#### Scenario: Role mismatch

- **WHEN** a request arrives with a valid session but the user's role is insufficient for the endpoint
- **THEN** the dependency guard SHALL raise `HTTPException(status_code=403)`

### Requirement: Session logout

The system SHALL allow authenticated users to terminate their browser session.

#### Scenario: User logs out

- **WHEN** an authenticated user clicks "Logout" or submits `POST /manage/logout`
- **THEN** the system SHALL respond with a `Set-Cookie` header that clears the session cookie and redirect to `/manage/login`

> **Note:** The server-side session data remains in memory but the cookie is cleared, rendering it effectively unusable from the browser.

### Requirement: RBAC enforcement on admin routes

The system SHALL enforce the same RBAC model as the CLI commands: admin has full access, team_lead is scoped to their team, developer and viewer roles SHALL NOT access `/manage` functionality.

#### Scenario: Developer and Viewer roles denied

- **WHEN** a user with `developer` or `viewer` role accesses any `/manage` page or API
- **THEN** the system SHALL return `403 Forbidden`
- **THEN** the page template SHALL render "Access denied"

#### Scenario: Team lead scope enforcement

- **WHEN** a user with `team_lead` role accesses user or key lists
- **THEN** the system SHALL only show data belonging to the team_lead's own team

#### Scenario: Admin full access

- **WHEN** a user with `admin` role accesses any `/manage` section
- **THEN** the system SHALL show all users, teams, keys, and usage data across all teams
