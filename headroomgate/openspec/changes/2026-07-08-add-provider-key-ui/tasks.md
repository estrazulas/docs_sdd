## 1. Backend: Store method for key removal

- [x] 1.1 Add `remove_provider_key(role_name, provider)` method to `Neo4jAuthStore` in `headroom/auth/store.py`
- [x] 1.2 Handle role-not-found case (raise `AuthStoreError`)
- [x] 1.3 Handle non-existent provider entry (no-op, still succeed)

## 2. Backend: Provider key API endpoints

- [x] 2.1 Implement `GET /manage/api/roles` — list all roles via `store.list_roles()`
- [x] 2.2 Implement `GET /manage/api/roles/{name}/providers` — list configured providers via `store.list_provider_keys()`
- [x] 2.3 Implement `POST /manage/api/roles/{name}/providers/{provider}` — set provider key via `store.set_provider_key()`, check `HEADROOM_ENCRYPTION_KEY` availability, validate provider name against allowed list
- [x] 2.4 Implement `DELETE /manage/api/roles/{name}/providers/{provider}` — remove provider key via `store.remove_provider_key()`
- [x] 2.5 Add RBAC enforcement (admin-only) and admin role protection (403 for admin role)

## 3. Frontend: Roles page template

- [x] 3.1 Create `roles.html` page with Alpine.js app — card grid showing all roles with name, description, provider count, protected badge
- [x] 3.2 Implement provider key modal — list configured providers, "Add Key" form (provider dropdown + password input), remove button with confirmation dialog
- [x] 3.3 Add JavaScript functions: `loadRoles()`, `loadProviders(role)`, `setProviderKey(role, provider, key)`, `removeProviderKey(role, provider)`
- [x] 3.4 Add page route `GET /manage/roles` and empty state/access_denied handling

## 4. Frontend: Sidebar navigation

- [x] 4.1 Add "Roles" menu link to `base.html` sidebar, visible only to admin users
- [x] 4.2 Add SVG icon for Roles menu item

## 5. Verification

- [x] 5.1 Start the proxy and verify the Roles page appears in the sidebar for admin
- [x] 5.2 Verify provider keys can be set and removed for a custom role
- [x] 5.3 Verify base roles (admin, developer, team_lead, viewer) show protected badge and reject modifications
- [x] 5.4 Verify team_lead and developer cannot access the Roles page
- [x] 5.5 Run lint and type checks
- [x] 5.6 Add E2E tests: roles page loads, protected badge, Keys buttons, modal opens, set/remove key flow, non-admin blocked
