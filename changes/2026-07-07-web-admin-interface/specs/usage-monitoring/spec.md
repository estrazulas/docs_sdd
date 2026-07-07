## ADDED Requirements

### Requirement: Top consumers chart

The system SHALL display a horizontal bar chart on the usage page showing the top consumers by token usage, filtered by a configurable time window (default: last 7 days).

#### Scenario: Top users by token usage

- **WHEN** the usage page loads
- **THEN** the system SHALL render a horizontal bar chart of users (or teams for team leads) ordered by total tokens consumed in the selected time window

#### Scenario: Time window switch

- **WHEN** the user selects "24h", "7d", or "30d" from the time window selector
- **THEN** the summary cards and chart SHALL update to reflect the selected period

### Requirement: User session drill-down

The system SHALL allow looking up a user's recent request history by typing a username or user ID, showing each session with timestamp, model, input/output tokens, tokens saved, and a text summary.

#### Scenario: Look up user sessions

- **WHEN** an admin types a username into the "User History" input and clicks "Load"
- **THEN** the system SHALL fetch that user's request history and display it in a table

#### Scenario: Session detail shows summary

- **WHEN** a user's sessions are displayed
- **THEN** each session row SHALL show the timestamp, model, token counts, and request summary

#### Scenario: Team lead restricted to own team

- **WHEN** a team lead views usage
- **THEN** the system SHALL only show sessions for users in their team

### Requirement: Semantic search

The system SHALL provide a search input that queries request history via Qdrant semantic search, returning matching sessions ranked by relevance.

#### Scenario: Search by term

- **WHEN** a user types a search term and submits
- **THEN** the system SHALL return sessions whose text summary matches the term, ranked by relevance score

#### Scenario: Search respects RBAC scope

- **WHEN** a team lead performs a search
- **THEN** the system SHALL only return results from users in the team lead's team

### Requirement: Usage summary metrics

The system SHALL display aggregate usage metrics at the top of the usage page: total tokens, total requests, active users, tokens saved, and estimated cost savings, filtered by the selected time window.

#### Scenario: Summary cards

- **WHEN** the usage page loads
- **THEN** the system SHALL display summary cards showing total tokens consumed, total requests, unique active users, tokens saved, and estimated cost savings in the selected time window

#### Scenario: Summary updates with time window

- **WHEN** the time window selector changes
- **THEN** all summary cards SHALL update to reflect the new period

### Requirement: RBAC for usage data

The system SHALL enforce scope-based access control on all usage queries. Team leads SHALL only see data for their own team; admins SHALL see all data.

#### Scenario: Developer and Viewer roles redirected

- **WHEN** a user with `developer` or `viewer` role accesses `/manage/usage`
- **THEN** the system SHALL display "Access denied" (only admins and team leads can access usage monitoring)
