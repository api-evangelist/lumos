---
name: lumos-offboard-user
description: Trigger or schedule a Lumos user offboarding, check its state, and cancel it while it is still scheduled or in progress.
api: Lumos REST API
base_url: https://api.lumos.com
api_reference: https://developers.lumos.com/reference/createuseroffboarding
generated: '2026-08-29'
method: generated
source: openapi/lumos-openapi.json, conventions/lumos-conventions.yml
operations:
  - listUsers
  - getUser
  - getUserAccounts
  - createUserOffboarding
  - getUserOffboarding
  - cancelUserOffboarding
---

# Offboard a user

This is the highest-consequence write in the Lumos contract: it removes a person's access across
every connected app. Read the reversal window before you call it.

## Steps

1. **Resolve the identity.** `GET /users` (`listUsers`) with a filter, then `GET /users/{user_id}`
   (`getUser`) to confirm you have the right person.
2. **See what will be revoked.** `GET /users/{user_id}/accounts` (`getUserAccounts`) lists the
   accounts in connected apps that the offboarding will act on. Do this before step 3 — there is no
   dry-run mode on the offboarding endpoint.
3. **Trigger or schedule it.** `POST /lifecycle-management/user-offboardings`
   (`createUserOffboarding`) with `user_id`. Returns `201`.
4. **Track it.** `GET /lifecycle-management/user-offboardings/{user_offboarding_id}`
   (`getUserOffboarding`).
5. **Cancel it if it was a mistake.** `POST /lifecycle-management/user-offboardings/{id}/cancel`
   (`cancelUserOffboarding`). The reference states this cancels "a scheduled or in-progress user
   offboarding" — that is the window. Once it is past that state the call returns `409` and the
   revocations must be undone one at a time by re-granting access.

## Rules

- **No dry run, no idempotency key.** Step 2 is the rehearsal, and a retried step 3 may create a
  second offboarding — check `getUserOffboarding` before retrying.
- `403` means the caller may not manage lifecycle on this domain. `409` on cancel means the window has
  closed. `404` means the offboarding id is not visible to this domain.
- Escalate to a human before calling step 3 in any automated flow.
