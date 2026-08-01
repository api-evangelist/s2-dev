---
name: Issue a scoped S2 access token
description: Mint a least-privilege S2 access token scoped to specific basins, streams, and operations, then revoke it.
api: openapi/s2-dev-openapi-original.json
operations: [issue_access_token, list_access_tokens, revoke_access_token]
---

# Issue a scoped S2 access token

Use this to hand an agent or service a least-privilege credential instead of a
root token.

## Auth
`Authorization: Bearer $S2_ACCESS_TOKEN` with account-level write permission to
manage tokens.

## Steps
1. **Issue.** Call `issue_access_token` (`POST /access-tokens`) with:
   - `id` (1-96 chars),
   - `scope`: an AccessTokenScope selecting `basins` / `streams` /
     `access_tokens` resource sets and `op_groups` (account/basin/stream
     read+write permissions),
   - optional `auto_prefix_streams` to namespace streams to a prefix,
   - optional `expires_at` (RFC 3339).
   The response returns the token string ONCE — store it in a secret manager.
2. **Audit.** Call `list_access_tokens` (`GET /access-tokens`) to review token
   metadata (prefix filter, cursor pagination via `has_more`).
3. **Revoke.** Call `revoke_access_token` (`DELETE /access-tokens/{id}`) when the
   token is no longer needed.

## Rules
- Grant only the resource sets and op_groups the workload needs (e.g. stream
  read-only for a live-view consumer).
- Prefer short `expires_at` for agent sessions.
- A 403 at use time means the token's scope excludes that operation/resource.
- See authentication/s2-dev-authentication.yml.
