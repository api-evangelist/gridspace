---
name: Send an SMS and read replies with Guava
description: Send an SMS from a Guava number and poll the inbox for the recipient's reply.
api: openapi/gridspace-guava-openapi.yml
operations: [sendSms, listMessages]
---

# Send an SMS and read replies with Guava

## Auth
`Authorization: Bearer YOUR_GUAVA_API_KEY` on every request. Base URL `https://api.goguava.ai/v1`.

## Prerequisites
Sending SMS requires completed SMS brand + campaign registration (A2P 10DLC) for your org, and the `from_number` must be a Guava number with SMS enabled. See https://goguava.ai/docs/outbound-and-sms-permissions.

## Steps
1. **Send** — `sendSms` (`POST /send-sms`) with JSON `{from_number, to_number, message}` (both numbers E.164). Success is `201` with `{"status":"sent"}`. `400` = from_number not owned/SMS-not-configured; `500` = upstream carrier rejected.
2. **Poll for the reply** — `listMessages` (`GET /messages`) with required `to_number` set to *your* Guava number (the inbox to read). Optionally narrow with `from_number` (the recipient) and `start` (ISO 8601). Messages return oldest-first with `has_more`.

## Rules
- Pagination is timestamp-windowed: page by setting `start` to the `received_at` of the last message. `start` is **inclusive**, so the boundary message repeats — dedupe by `id`.
- URL-encode `+` in query strings as `%2B` (or use `curl -G --data-urlencode`).
- No idempotency-key mechanism exists; avoid duplicate sends at the application layer.
