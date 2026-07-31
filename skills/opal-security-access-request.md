---
name: opal-access-request
description: Request, review, and grant just-in-time access to an Opal-managed resource or group via the Opal API. Use when a user needs temporary or standing access, when an approver must action a pending request, or when an admin grants access directly.
api: Opal API
base_url: https://api.opal.dev/v1
auth: Bearer token (service-user API key or personal access token) in the Authorization header
operations:
- getResources
- get_resource
- getGroups
- createRequest
- getRequests
- getRequest
- approveRequest
- denyRequest
- get_resource_users
- add_resource_user
generated: '2026-07-20'
method: generated
source: openapi/opal-security-openapi.yaml
---

# Opal — Access Request Lifecycle

Grounded in real Opal API operationIds. All calls go to `https://api.opal.dev/v1`
with `Authorization: Bearer <OPAL_API_TOKEN>`. List endpoints are cursor-paginated
(`cursor`, `page_size`). Errors are plain-JSON with standard HTTP codes; `403`
means the token's Opal role lacks permission, `404` means the entity isn't visible
to the caller.

## 1. Find the target
- List resources with `getResources` (`GET /resources`) or groups with `getGroups`
  (`GET /groups`); page with `cursor` until you find the `resource_id` / `group_id`.
- Fetch details with `get_resource` (`GET /resources/{resource_id}`) to confirm the
  `admin_owner_id` and request configuration.

## 2. Create the access request
- Call `createRequest` (`POST /requests`) with the target user/group and the
  resource or group to request, plus any required duration/justification fields.
- The request routes to the configured reviewers/owners for approval.

## 3. Track and review
- Poll `getRequests` (`GET /requests`) or `getRequest` (`GET /requests/{id}`) to
  read status.
- An approver actions the request with `approveRequest`
  (`POST /requests/{id}/approve`) or `denyRequest` (`POST /requests/{id}/deny`).

## 4. Direct grant (admin path)
- To bypass the request flow, an admin can grant directly with `add_resource_user`
  (`POST /resources/{resource_id}/users`) and confirm with `get_resource_users`
  (`GET /resources/{resource_id}/users`).

## Notes
- Use a **full-access** personal access token or a service user with the required
  scoped Opal role; **read-only** tokens can only perform GET.
- Prefer just-in-time, time-bound requests over standing direct grants to preserve
  least-privilege posture.
