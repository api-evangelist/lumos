---
name: lumos-run-access-review
description: Create a Lumos access review campaign, add applications to it, watch it move out of IN_PREPARATION, and delete it or an app from it if the scope was wrong.
api: Lumos REST API
base_url: https://api.lumos.com
api_reference: https://developers.lumos.com/reference/createaccessreview
generated: '2026-08-29'
method: generated
source: openapi/lumos-openapi.json, https://developers.lumos.com/llms.txt
operations:
  - createAccessReview
  - addAppsToAccessReview
  - getAccessReview
  - listAccessReviews
  - updateAccessReview
  - getScopeOptions
  - deleteAccessReviewApp
  - deleteAccessReview
---

# Run an access review campaign

## Steps

1. **Check the scope filters available.** `GET /access_reviews/scope_options` (`getScopeOptions`)
   returns the filter options valid for a domain app or ARDA. Use these values in `scope_filters`;
   do not construct them yourself.
2. **Create the campaign.** `POST /access_reviews` (`createAccessReview`) with `owner_id`.
   - **With apps supplied**, the review starts in `IN_PREPARATION` while account and entitlement
     snapshots are taken, then moves itself to `IN_PROGRESS`.
   - **Without apps**, it is created directly in `IN_PROGRESS`.
3. **Poll for the transition.** `GET /access_reviews/{access_review_id}` (`getAccessReview`) until the
   status leaves `IN_PREPARATION`. There is no webhook for this — polling is the documented method.
4. **Add apps later if needed.** `POST /access_reviews/{access_review_id}/apps`
   (`addAppsToAccessReview`) is allowed on any review that is not `COMPLETED`. It soft-errors per app:
   read `AccessReviewDomainAppErrorOutput` entries in the response for apps that could not be added,
   each with an `app_id` and a reason.
5. **Correct the scope.** `DELETE /access_reviews/{access_review_id}/apps/{arda_id}`
   (`deleteAccessReviewApp`) soft-deletes one app from the campaign; it fails with `400` if the review
   is already `COMPLETED`.
6. **Cancel the campaign.** `DELETE /access_reviews/{access_review_id}` (`deleteAccessReview`)
   soft-deletes the whole review. It fails with `400` while a review duplication is in progress.

## Rules

- `arda_id` is the id of the AccessReviewDomainApp join, **not** the app's id. Passing `app_id` here
  returns `404`.
- Both deletes are soft, and both are the documented reversal path — see
  `conventions/lumos-conventions.yml`.
- `listAccessReviews` paginates with `page` + `size`; `size` is capped by the server.
