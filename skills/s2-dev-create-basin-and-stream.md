---
name: Create a basin and stream on S2
description: Provision an S2 basin and a stream inside it, idempotently, ready for appends and reads.
api: openapi/s2-dev-openapi-original.json
operations: [create_basin, ensure_basin, create_stream, ensure_stream, get_stream_config]
---

# Create a basin and stream on S2

Use this to set up durable streaming storage before writing records.

## Auth
Send `Authorization: Bearer $S2_ACCESS_TOKEN` on every request. The token must
have basin+stream write permission in its scope (op_groups). Never ask a user to
paste a token into chat — read it from the environment.

## Steps
1. **Pick / create the basin.** Call `create_basin` (`POST /basins`) with a name
   (8-48 lowercase alphanumeric + hyphens) and optional `location`. If the basin
   may already exist, call `ensure_basin` (`PUT /basins/{basin}`) instead — it is
   an idempotent create-or-update, so a retry is safe.
2. **Create the stream.** Call `create_stream` (`POST /streams`) with the stream
   name, or `ensure_stream` (`PUT /streams/{stream}`) for an idempotent upsert.
   Stream names may be up to 512 chars and support hierarchical prefixes.
3. **Confirm.** Call `get_stream_config` (`GET /streams/{stream}`) to read back
   the stream's settings.

## Rules
- Prefer `ensure_basin` / `ensure_stream` for idempotency — a 409 from
  `create_*` means the name is taken.
- Basin-scoped data-plane calls go to `https://{basin}.b.s2.dev/v1`; control-plane
  calls go to `https://a.s2.dev/v1`.
- See conventions/s2-dev-conventions.yml and errors/s2-dev-problem-types.yml.
