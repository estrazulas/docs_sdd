## ADDED Requirements

### Requirement: List API keys

The system SHALL display all API keys with their prefix (`hr_*`), associated user, active/expired/revoked status, creation date, and expiration date.

#### Scenario: Admin lists all keys

- **WHEN** an admin navigates to "API Keys"
- **THEN** the system SHALL display all keys across all users with status indicators

#### Scenario: Team lead lists team keys

- **WHEN** a team lead navigates to "API Keys"
- **THEN** the system SHALL only display keys belonging to users in their team

### Requirement: Create API key

The system SHALL provide a form to generate a new `hr_*` API key for a selected user with a configurable TTL (default 90 days). Upon creation, the full key SHALL be displayed once with a copy-to-clipboard button and JSON output format for distribution.

#### Scenario: Admin creates a key for a user

- **WHEN** an admin selects a user, optionally sets a TTL, and submits
- **THEN** the system SHALL call `Neo4jAuthStore.create_key()`, display the full key once with a copy button, and provide a JSON snippet with setup instructions

#### Scenario: Key displayed only once

- **WHEN** the generated key is displayed and the user navigates away or refreshes
- **THEN** the full key SHALL NOT be retrievable again (only the prefix and metadata remain visible)

#### Scenario: Copy-to-clipboard JSON output

- **WHEN** a key is generated
- **THEN** the system SHALL display a JSON block containing the API key and configuration instructions that can be copied with a single click

### Requirement: Revoke API key

The system SHALL allow revoking an active API key with a confirmation prompt.

#### Scenario: Admin revokes a key

- **WHEN** an admin clicks "Revoke" on an active key and confirms
- **THEN** the system SHALL call `Neo4jAuthStore.revoke_key()` and update the key status to "revoked"

#### Scenario: Revoked key cannot be undone

- **WHEN** a key is revoked
- **THEN** the system SHALL display a warning before revocation that the action is irreversible
