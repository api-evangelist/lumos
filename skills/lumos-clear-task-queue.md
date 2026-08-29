---
name: lumos-clear-task-queue
description: Work the Lumos Task Center — find approval, provisioning and error tasks, inspect the actions available on each, and complete, dismiss, reassign or comment on them.
api: Lumos REST API
base_url: https://api.lumos.com
api_reference: https://developers.lumos.com/reference/list_tasks_tasks_get
generated: '2026-08-29'
method: generated
source: openapi/lumos-openapi.json, mcp/lumos-tool-crosswalk.yml
operations:
  - list_tasks_tasks_get
  - get_task_tasks__task_id__get
  - get_task_actions_tasks__task_id__actions_get
  - complete_task_tasks__task_id__complete_post
  - perform_task_action_tasks__task_id__perform_action_post
  - dismiss_task_tasks__task_id__dismiss_post
  - reassign_task_tasks__task_id__reassign_post
  - add_task_comment_tasks__task_id__comments_post
---

# Work the Lumos Task Center

## Steps

1. **Find the tasks.** `GET /tasks` (`list_tasks_tasks_get`). Combine `task_category`,
   `product_area` and `status` to narrow the queue — e.g. `task_category=APPROVAL` and
   `product_area=APPSTORE` for access-request approvals.
2. **Read one.** `GET /tasks/{task_id}` (`get_task_tasks__task_id__get`).
3. **Discover what you may do to it.** `GET /tasks/{task_id}/actions`
   (`get_task_actions_tasks__task_id__actions_get`). Do this before acting — the available actions are
   task-specific and are the only safe input to step 4.
4. **Act.** Pick exactly one:
   - `POST /tasks/{task_id}/complete` (`complete_task...`) — the approve primitive. Resolves the
     task's unique COMPLETED-transition action. No request body. Use only when step 3 returned exactly
     one completing action.
   - `POST /tasks/{task_id}/perform-action` (`perform_task_action...`) — the escape hatch when a task
     exposes several actions, or a custom workflow action.
   - `POST /tasks/{task_id}/dismiss` (`dismiss_task...`) — the deny primitive.
   - `POST /tasks/{task_id}/reassign` (`reassign_task...`) — hand it to other `user_ids` or `group_ids`.
   - `POST /tasks/{task_id}/comments` (`add_task_comment...`) — add context without changing state.

## Rules

- **Completing an approval task applies a real access grant. It cannot be undone by this API.** There
  is no un-complete; the only inverse is a separate revocation. Escalate to a human before automating
  step 4 on `task_category=APPROVAL`.
- **`409` is your friend on retry.** The contract states a `409 Conflict` here means the task is
  already in the requested status and is "safe to treat as idempotent". That is the correct recovery
  from a timed-out complete or dismiss — do not re-drive the workflow.
- `403` means the caller is not an assignee of the task and not a domain admin.
- `404` can also mean the Tasks API is not enabled for the domain, not just that the task is missing.
