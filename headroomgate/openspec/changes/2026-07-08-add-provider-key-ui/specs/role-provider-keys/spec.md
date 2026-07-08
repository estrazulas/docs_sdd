## ADDED Requirements

### Requirement: Role list page

The system SHALL display a list of all roles on a dedicated `/manage/roles` page accessible only to admin users. Each role entry SHALL show the role name, description, a count of configured providers, and a visual indicator if it is a protected base role.

#### Scenario: Admin views role list
- **WHEN** an admin user navigates to `/manage/roles`
- **THEN** the system SHALL display a table or card grid with all roles (name, description, provider count, protected badge)
- **AND** the "Roles" menu item SHALL be highlighted in the sidebar

#### Scenario: Team lead cannot access roles
- **WHEN** a team_lead user navigates to `/manage/roles`
- **THEN** the system SHALL return 403 Forbidden or redirect to access denied

#### Scenario: Developer cannot access roles
- **WHEN** a developer user navigates to `/manage/roles`
- **THEN** the system SHALL return 403 Forbidden or redirect to access denied

#### Scenario: Admin sees protected badge on admin role only
- **WHEN** an admin user views the role list
- **THEN** the `admin` role SHALL display a "Protected" badge
- **AND** the `admin` role SHALL NOT show a provider key management action icon
- **AND** all other roles (`developer`, `team_lead`, `viewer`, custom roles) SHALL show the gear icon for key management

#### Scenario: Roles page shows empty providers count
- **WHEN** an admin user views the role list
- **THEN** each role SHALL display the number of configured providers (e.g., "2 providers" or "No providers")

### Requirement: View provider keys for a role

The system SHALL allow an admin user to view which provider API keys are configured for a specific role, without revealing the key values.

#### Scenario: Admin views providers for a custom role
- **WHEN** an admin user clicks the provider key icon on a custom role
- **THEN** the system SHALL open a modal listing configured providers (Anthropic, OpenAI, Gemini) with status "configured" or "not configured"
- **AND** the actual key values SHALL NOT be displayed (only status)

#### Scenario: Admin views providers for a protected role
- **WHEN** an admin user interacts with the `admin` role card
- **THEN** the system SHALL not show the gear action icon, and display a "Protected" badge instead

### Requirement: Set a provider key

The system SHALL allow an admin user to set a provider API key for a custom role, encrypting it with Fernet before storage.

#### Scenario: Admin sets Anthropic key for a custom role
- **WHEN** an admin user selects "Add Key" on a custom role, chooses "Anthropic" from the provider dropdown, enters a valid API key, and submits
- **THEN** the system SHALL encrypt the key with `HEADROOM_ENCRYPTION_KEY` via `FernetCrypto`
- **AND** store it in the role's `provider_keys` JSON dict in Neo4j
- **AND** display the provider as "configured" in the UI

#### Scenario: Admin sets key with missing encryption key
- **WHEN** an admin user attempts to set a provider key but `HEADROOM_ENCRYPTION_KEY` is not configured
- **THEN** the system SHALL return a 400 error with a message guiding the admin to run `headroom auth generate-key`

#### Scenario: Admin tries to set key on admin role
- **WHEN** an admin user attempts to set a provider key for the `admin` role
- **THEN** the system SHALL return 403 Forbidden

#### Scenario: Admin sets key with empty value
- **WHEN** an admin user submits the provider key form with an empty key value
- **THEN** the system SHALL return 400 Bad Request

### Requirement: Remove a provider key

The system SHALL allow an admin user to remove a provider API key from a custom role, deleting the encrypted entry from the role's `provider_keys` JSON dict.

#### Scenario: Admin removes provider key from custom role
- **WHEN** an admin user clicks the remove icon on a configured provider for a custom role
- **AND** confirms the removal in a confirmation dialog
- **THEN** the system SHALL remove the provider entry from the role's `provider_keys` JSON dict
- **AND** display the provider as "not configured" in the UI

#### Scenario: Admin tries to remove key from admin role
- **WHEN** an admin user attempts to remove a provider key from the `admin` role
- **THEN** the system SHALL return 403 Forbidden

#### Scenario: Confirmation before removal
- **WHEN** an admin user clicks remove on a configured provider key
- **THEN** the system SHALL display a confirmation dialog ("Remove {provider} key from {role}? This cannot be undone.")
- **AND** only proceed with removal upon confirmation

### Requirement: Sidebar navigation

The system SHALL display a "Roles" link in the admin sidebar navigation, visible only to admin users.

#### Scenario: Admin sees Roles in sidebar
- **WHEN** an admin user is logged into the admin interface
- **THEN** the sidebar SHALL show a "Roles" menu item with an appropriate icon
- **AND** clicking it SHALL navigate to `/manage/roles`

#### Scenario: Non-admin does not see Roles in sidebar
- **WHEN** a team_lead or developer user is logged into the admin interface
- **THEN** the sidebar SHALL NOT show the "Roles" menu item

### Requirement: Backend store supports provider key removal

The system SHALL implement a `remove_provider_key(role_name, provider)` method in `Neo4jAuthStore` that removes a single provider entry from the role's `provider_keys` JSON dict.

#### Scenario: Remove existing provider key
- **WHEN** `store.remove_provider_key("custom_role", "anthropic")` is called
- **AND** the role has an "anthropic" entry in its `provider_keys`
- **THEN** the "anthropic" entry SHALL be removed from the JSON dict
- **AND** the method SHALL return a dict indicating the removal

#### Scenario: Remove non-existent provider key
- **WHEN** `store.remove_provider_key("custom_role", "nonexistent")` is called
- **AND** the role has no "nonexistent" entry
- **THEN** the method SHALL still succeed (no-op), returning an empty result

#### Scenario: Remove from non-existent role
- **WHEN** `store.remove_provider_key("nonexistent_role", "anthropic")` is called
- **THEN** the method SHALL raise `AuthStoreError` with a message that the role does not exist
