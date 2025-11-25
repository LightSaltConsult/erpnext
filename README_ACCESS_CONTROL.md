# ERPNext Access Control Overview and Implementation Guide

This guide explains how ERPNext's access control works and how you can adopt the same patterns in your own Frappe/ERPNext-based apps. It covers the three main layers—role-based permissions, value-level user permissions, and ad-hoc sharing—plus a practical rollout checklist and implementation tips.

## Layers of access control

### 1. Role-based permissions
ERPNext ships with the Role Permissions Manager, which lets you define CRUD and workflow permissions per DocType and permission level (permlevel). The help links exposed in the UI point directly to the Role-Based Permissions documentation and the Role Permissions Manager, underscoring that role definitions are the foundation of every permission decision.【F:erpnext/public/js/help_links.js†L25-L75】

### 2. Value-level user permissions
When you need to scope access to specific records (e.g., a subset of Departments), ERPNext uses the `User Permission` DocType. Records in `tabUser Permission` store the target DocType (`allow`) and the specific record value (`for_value`). The V11 patch `set_user_permissions_for_department.py` demonstrates this structure by copying existing User Permission records and applying them to each company-specific department, showing how value filters are persisted and reused programmatically.【F:erpnext/patches/v11_0/set_user_permissions_for_department.py†L5-L23】

### 3. Ad-hoc sharing
For exceptions that don't fit cleanly into roles or value filters, ERPNext surfaces a Sharing dialog in the UI. The help links list Sharing alongside role and user permissions, highlighting that specific documents can be shared with another user for read or write access without changing their global roles.【F:erpnext/public/js/help_links.js†L25-L43】

## End-to-end permission evaluation flow
1. **Baseline via roles**: The permission engine first checks whether the user has a role that grants the requested action (read, write, create, submit, delete) at the relevant permlevel for the DocType.
2. **Scope via user permissions**: If role access is present, the system narrows the allowed records by enforcing any `User Permission` filters tied to that user (for example, restricting Department access to specific rows in `tabDepartment`).
3. **Overrides via sharing**: Finally, if a document has been explicitly shared with the user, that share can grant access even if the user would otherwise be blocked by user permissions, depending on the share's permission flags (read, write, submit, share).
4. **System settings and hooks**: Global toggles (e.g., permissions based on document ownership) and server hooks (e.g., custom permission queries) can further refine access as needed for your app.

## Implementing this model in your own app

### Prepare your DocTypes
- **Assign meaningful permlevels**: Use permlevel 0 for core fields and higher permlevels for sensitive fields. This allows you to grant granular access in the Role Permissions Manager.
- **Add link fields for scoping**: Include `Link` fields (e.g., Company, Department, Territory) that you can target with `User Permission` filters.

### Configure role-based rules
1. Create roles that map to business personas (e.g., Sales User, HR Manager).
2. Open **Role Permissions Manager** and set permissions per DocType and permlevel (read, write, create, submit, delete, set user permissions).
3. Assign roles to users via the User master.

### Add user-level scoping
1. Create `User Permission` records for each user and link field you want to filter (e.g., allow = Department, for_value = "North Division").
2. Where you have multi-company or multi-division setups, script initialization patches similar to `set_user_permissions_for_department.py` to replicate permissions across new records automatically.【F:erpnext/patches/v11_0/set_user_permissions_for_department.py†L5-L23】
3. Test by logging in as the scoped user and verifying that list views and link field options are restricted accordingly.

### Support sharing scenarios
1. Enable users with `Share` rights to open a document and share it with another user, specifying read/write/submit/share flags.
2. Use sharing for temporary or exception-based access rather than replacing role or user-permission setups.

### Operational checklist for rollout
- [ ] Map business personas to roles and decide which DocTypes each persona can see or modify.
- [ ] Identify key scoping fields (e.g., Company, Department) and ensure they exist on your DocTypes.
- [ ] Configure Role Permissions Manager for each DocType/permlevel.
- [ ] Seed user permissions for scoped fields and automate propagation when new records are created.
- [ ] Validate with representative user accounts and refine as needed.

### Best practices
- Keep roles stable and descriptive; avoid one-off roles for edge cases—use sharing instead.
- Review and prune `User Permission` records periodically to prevent stale scoping rules.
- Treat automation scripts (patches or scheduled jobs) as the source of truth for bulk permission updates to maintain auditability.【F:erpnext/patches/v11_0/set_user_permissions_for_department.py†L5-L24】
- Document your permission model so new maintainers understand where to adjust roles, user permissions, and sharing.

## Quick start summary
1. Use **Role Permissions Manager** to set CRUD rights per DocType and permlevel.【F:erpnext/public/js/help_links.js†L25-L75】
2. Add `User Permission` records to filter data scope for specific users.【F:erpnext/patches/v11_0/set_user_permissions_for_department.py†L5-L23】
3. Rely on document **Sharing** for exceptions or temporary access.【F:erpnext/public/js/help_links.js†L25-L43】

Following these steps gives you a layered permission model that is easy to audit, automatable for scale, and flexible enough to handle exceptions.
