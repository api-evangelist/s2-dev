---
name: Append records and tail a stream on S2
description: Write records atomically to an S2 stream and read/tail them in real time.
api: openapi/s2-dev-openapi-original.json
operations: [append, read, check_tail]
---

# Append records and tail a stream on S2

Use this for the S2 data plane: writing events and following them live.

## Auth
`Authorization: Bearer $S2_ACCESS_TOKEN`; the token needs stream read/write
permission. Data-plane host is `https://{basin}.b.s2.dev/v1`.

## Append
1. Call `append` (`POST /streams/{stream}/records`) with a batch of records.
   Limits: at most 1000 records or 1 MiB per batch; a record is at most 1 MiB.
2. For concurrency safety pass an append condition:
   - `match_seq_num` — optimistic: the append only succeeds if the tail is at the
     expected sequence number (else HTTP 412).
   - `fencing_token` — pessimistic: a stale token is rejected (HTTP 412).
3. JSON record bodies are bytes: use raw UTF-8 by default, or set the
   `s2-format: base64` header for arbitrary binary. `Content-Type:
   application/protobuf` is also supported.

## Read / tail
4. Call `read` (`GET /streams/{stream}/records`) with `seq_num`, a timestamp, or
   a tail-relative offset, plus `count`. Add `?wait=<secs>` for long-poll tailing,
   or use SSE (`Accept: text/event-stream`) in browser contexts.
5. Call `check_tail` (`GET /streams/{stream}/records/tail`) to get the current end
   position for coordination / freshness checks.

## Rules
- The tail is the position AFTER all current records; reading "from the tail"
  means "start after everything that exists now."
- Over-limit appends return HTTP 429; append sessions are throttled instead.
- 416 means the requested range is outside the stream's retained window.
- See conventions/s2-dev-conventions.yml, rate-limits/s2-dev-rate-limits.yml.
