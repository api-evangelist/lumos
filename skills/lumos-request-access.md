---
name: lumos-request-access
description: Request access to an application permission in the Lumos AppStore on behalf of a user, then track or cancel that request.
api: Lumos REST API
base_url: https://api.lumos.com
api_reference: https://developers.lumos.com/reference/lumos-api
generated: '2026-08-29'
method: generated
source: openapi/lumos-openapi.json, conventions/lumos-conventions.yml, errors/lumos-error-codes.yml
operations:
  - getAppStoreApps
  - get_appstore_permissions_for_app_appstore_apps__app_id__requestable_permissions_get
  - createAccessRequest
  - getAccessRequest
  - getAccessRequests
  - cancelAccessRequest
---

# Request access in the Lumos AppStore

Authenticate every call with `Authorization: Bearer lsk_...`. The MCP servers use OAuth instead —
an API key will not work there.

## Steps

1. **Find the app.** `GET /appstore/apps` (`getAppStoreApps`). Filter with `name_search` and
   `exact_match`. Paginate with `page` and `size`; the server caps `size`.
2. **List what can be requested on it.** `GET /appstore/apps/{app_id}/requestable_permissions`
   (`get_appstore_permissions_for_app_appstore_apps__app_id__requestable_permissions_get`). Collect the
   permission `id` values you need.
3. **Submit the request.** `POST /appstore/access_request` (`createAccessRequest`) with `app_id`,
   `requestable_permission_ids`, and `business_justification`. Set `target_user_id` to request on
   someone else's behalf; omit it to request for the current user. Optionally set
   `expiration_in_seconds` for time-bound access.
   - Do **not** use the `note` field. The contract marks it `[Deprecated - use business_justification]`.
4. **Track it.** `GET /appstore/access_requests/{id}` (`getAccessRequest`), or list with
   `GET /appstore/access_requests` (`getAccessRequests`).
5. **Reverse it if needed.** `DELETE /appstore/access_requests/{id}` (`cancelAccessRequest`) cancels a
   request while it is still pending. This is the only undo — once an approval task is completed the
   grant has been applied and must be revoked separately.

## Rules

- **There is no idempotency key.** Retrying step 3 creates a SECOND access request. Before retrying a
  timed-out `createAccessRequest`, call `getAccessRequests` and check whether the first one landed.
- **Errors.** Branch on `code` in the `ApiError` envelope, not on `detail`. `422` is validation
  (also possible in FastAPI's `detail[].loc` shape), `403` means the caller may not act on this
  domain, `429` carries `Retry-After` in seconds.
- **Rate limits.** Honour `x-ratelimit-remaining` and `retry-after` on every response.
