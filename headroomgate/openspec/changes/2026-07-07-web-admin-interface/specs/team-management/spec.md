## ADDED Requirements

### Requirement: List teams

The system SHALL display all teams with their member count. Only admins can access the Teams page.

#### Scenario: Admin lists all teams

- **WHEN** an admin navigates to "Teams"
- **THEN** the system SHALL display all teams with member counts

#### Scenario: Team lead cannot access teams page

- **WHEN** a team lead navigates to "Teams"
- **THEN** the system SHALL return `403 Forbidden` (team leads cannot create or manage teams)

### Requirement: Create team

The system SHALL provide a form to create a new team. Only admins can create teams.

#### Scenario: Admin creates a team

- **WHEN** an admin enters a team name and submits
- **THEN** the system SHALL call `Neo4jAuthStore.create_team()` and display the new team

### Requirement: Add user to team

The system SHALL allow admins to add a user to a team.

#### Scenario: Admin adds a user to a team

- **WHEN** an admin clicks "+ Add User" on a team card, enters a username, and submits
- **THEN** the system SHALL call `Neo4jAuthStore.add_user_to_team()` and update the team's member list
