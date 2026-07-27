---
name: Pull a Guava call record
description: Retrieve a completed Guava call's metadata, transcript, and audio recording via the REST API.
api: openapi/gridspace-guava-openapi.yml
operations: [listConversations, getConversation, getConversationTranscript, getConversationRecording]
---

# Pull a Guava call record

Use the Guava Voice Agent REST API to locate a call and download its full record.

## Auth
Every request needs `Authorization: Bearer YOUR_GUAVA_API_KEY` (key is prefixed `gva-`, created at https://app.goguava.ai/dashboard/api-keys). Base URL: `https://api.goguava.ai/v1`.

## Steps
1. **Find the call** — `listConversations` (`GET /conversations`). Filter with `from_number`/`to_number` (E.164 — URL-encode `+` as `%2B`), `direction`, `date_from`/`date_to`, or `campaign_id`. Results are newest-first; page with `limit` (1–100) and the `after` cursor until `has_more` is false. Grab the `call_id`.
2. **Get details** — `getConversation` (`GET /conversations/{call_id}`) for `direction`, `duration_sec` (null until the call completes), and `termination_reason`.
3. **Get the transcript** — `getConversationTranscript` (`GET /conversations/{call_id}/transcript`) returns turns of `{speaker: HUMAN|AGENT, text, offset_ms}`.
4. **Get the recording** — `getConversationRecording` (`GET /conversations/{call_id}/recording`) returns a WAV file; save with `-o recording.wav`.

## Rules
- Errors are plain JSON keyed by HTTP status: 401 = bad/missing key, 404 = unknown `call_id`, 422 = malformed E.164 (see errors/gridspace-problem-types.yml).
- No idempotency key is needed — these are read-only GETs.
- To delete a call and its data afterward, use `deleteConversation` (`DELETE /conversations/{call_id}`) — this is permanent.
