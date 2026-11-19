**Date:** 2026-11-19
**Environment:** Azure free trial, Microsoft Entra ID (cloud-only tenant)
**Skills:** Entra ID, Azure RBAC, least privilege, access troubleshooting, sign-in and audit log analysis

## Objective
Create a test user, grant read-only access through a security group, validate it, and trace the activity in the logs.

## Setup
- User: ana.analyst (Department: SOC)
- Group: SG-SOC-Analysts (Security, Assigned)
- Role assignment: Reader on rg-lab01-identity (scope: resource group)

## Issue found
- **Symptom:** user could not see any resource group
- **Investigation:** IAM "Check access" showed 0 role assignments
- **Root cause:** group created with 0 members
- **Fix:** added user to the group, then sign-out/sign-in (token refresh)
- **Validation:** user sees only rg-lab01-identity

## Least privilege test
- Tag write attempt → `AuthorizationFailed` (authorized to read, not write)

## Log analysis
- **Audit log:** "Add user", actor = admin, target = ana.analyst
- **Sign-in logs:** 2x Interrupted (expired temporary password, MFA registration) → Success
- Times in UTC; IP geolocation approximate
- **Activity log:** denied tag write not visible after ~15 min (possible logging gap, to verify)

## Key takeaways
- AuthN vs AuthZ
- Assign permissions to groups, not users
- Always validate access after granting it

## Evidence (sanitized)
| Log | Field | Value |
|---|---|---|
| Audit log | Activity | Add user |
| Audit log | Actor | admin@[tenant] |
| Audit log | Target | ana.analyst@[tenant] |
| Sign-in log | Status sequence | Interrupted → Interrupted → Success |
| Sign-in log | Interrupt reason | Expired temporary password |
| Sign-in log | IP / location | [REDACTED] / [city redacted] |
| Error | Code | AuthorizationFailed |