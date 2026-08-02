---
name: Run a Teammates assignment
description: Enqueue a natural-language assignment against a Smart Tool and retrieve its result, using the Teammates SmartTools API async enqueue-and-poll pattern.
api: openapi/teammates-smarttools-openapi.yaml
operations:
  - POST /assign
  - GET /assignment
---

# Run a Teammates assignment

Use the Teammates SmartTools API (`https://api.teammates.work/v1`) to have an AI
teammate perform a task against a connected Smart Tool using natural language.

## Steps

1. **Enqueue the assignment** — `POST /assign` with a JSON body:
   - `tool` (required): one of the supported Smart Tools (e.g. `google_sheets`,
     `github`, `salesforce`, `slack`, `jira`, `gmail`, `hubspot`, ...).
   - `prompt` (required): a natural-language description of the task, e.g.
     `"Update issue with ID 4 to status OK"`.
   - `data` (optional): structured data the task needs, e.g. `{ "issue_id": 4 }`.
   - `webhook_url` (optional): a URL to receive a status update on completion.
   The response returns `{ "assignment_id": "<uuid>" }`. Save the `assignment_id`.

2. **Retrieve the result** — poll `GET /assignment?assignment_id=<uuid>`.
   The response includes `status`, `duration`, and (when done) `result` +
   `details`. Terminal statuses: `complete`, `failed`, `canceled`.

3. **Handle non-terminal statuses** — if `status` is `in_progress` or
   `unstarted`, wait and poll again. If `clarification_required` or
   `waiting_for_others`, a human-in-the-loop response is needed before the
   assignment can continue. Prefer the `webhook_url` callback over tight polling.

## Notes

- The API is asynchronous: never expect a result from `/assign` — always read it
  from `/assignment` (or the webhook).
- The published spec declares no idempotency key; avoid re-submitting the same
  assignment on transient errors without checking existing `assignment_id`s.
- Give each teammate its own Smart Tool credentials so it operates as itself.
