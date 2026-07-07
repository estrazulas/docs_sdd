## ADDED Requirements

### Requirement: List users

The system SHALL display a paginated list of users with their username, role, team, and active status. Team leads SHALL only see users in their own team.

#### Scenario: Admin lists all users

- **WHEN** an admin navigates to "Users"
- **THEN** the system SHALL display all users across all teams with their details

#### Scenario: Team lead lists team users

- **WHEN** a team lead navigates to "Users"
- **THEN** the system SHALL only display users belonging to the team lead's team

### Requirement: Create user

The system SHALL provide a form to create a new user with username, role, and team fields. The form SHALL validate that the username is unique.

#### Scenario: Admin creates a user

- **WHEN** an admin fills in username, selects a role and team, and submits the form
- **THEN** the system SHALL call `Neo4jAuthStore.create_user()` and display the new user in the list

#### Scenario: Team lead creates a user in their team

- **WHEN** a team lead creates a user, the team field SHALL be pre-filled and locked to their own team
- **THEN** the system SHALL create the user scoped to that team

#### Scenario: Duplicate username rejected

- **WHEN** a user submits a username that already exists
- **THEN** the system SHALL return `409 Conflict` and SHALL NOT create the user

### Requirement: Deactivate user

The system SHALL allow deactivating a user, which also sets all their API keys as inactive.

#### Scenario: Admin deactivates a user

- **WHEN** an admin clicks "Deactivate" on an active user
- **THEN** the system SHALL call `Neo4jAuthStore.update_user_status(is_active=False)`, mark all keys as inactive, and show the user as inactive

### Requirement: Reactivate user

The system SHALL allow reactivating a previously deactivated user.

#### Scenario: Admin reactivates a user

- **WHEN** an admin clicks "Reactivate" on an inactive user
- **THEN** the system SHALL call `Neo4jAuthStore.update_user_status(is_active=True)` and show the user as active

### Requirement: Team lead cannot create users outside their team

The system SHALL reject user creation requests where the team lead tries to create a user in a team different from their own.

#### Scenario: Cross-team creation blocked

- **WHEN** a team lead submits a user creation form with a team different from their own
- **THEN** the system SHALL return `403 Forbidden` with detail "Cannot create user outside your team"
