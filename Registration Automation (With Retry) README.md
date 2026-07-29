# Registration Automation (With Retry)

This repository contains an n8n workflow named "Registration Automation (With Retry)" (file: `Registration Automation (With Retry).json`).

The workflow automates user registration by driving a browserless function to perform registration in a headless browser, records the result to Supabase, and retries on HTTP 429 responses using exponential backoff.

## Overview

- Trigger: Webhook — receives registration payloads (email, password).
- Generates a session id and records submission time.
- Calls Browserless to run a headless browser script that performs the registration flow.
- Tracks the status code and retry count.
- If the Browserless call returns HTTP 429 (rate limit), the workflow will retry up to a configurable limit with exponential backoff.
- On success, it inserts a registration record and sends a success email.
- On final failure, it inserts a failed registration record with error details.

## Files

- `Registration Automation (With Retry).json` — n8n workflow JSON export. Import this file into your n8n instance.
- `README.md` — this file.

## Nodes (high level)

- Webhook1 — receives POST requests to start a registration run. Expects JSON body with at least `email` and `password`.
- Generate Session ID1 — creates a `sessionId`, stores `email`, `password`, `submittedAt`, and initializes `retryCount` to 0.
- Register via Browserless1 — POST to Browserless function URL to run automated registration in a headless browser. The `code` payload runs the page interactions. The node is configured to return the full response and not to error on non-2xx responses.
- Track Retry Count — picks up the `statusCode` from the Browserless response and the current `retryCount` (either from the Increment node if retried, or the generated session id on first try).
- Check Status (429?) — checks whether the returned `statusCode` equals `429`. If yes, flow goes into the retry branch.
- Retries Remaining? — checks if `retryCount` is less than the configured threshold (currently 5). If true, it continues to backoff and retry.
- Wait (Backoff) — waits `5 * 2^retryCount` seconds (exponential backoff) before attempting again.
- Increment Retry Count — increments the `retryCount` and re-calls Register via Browserless1.
- Insert Registration Record2 — on success, inserts a record into Supabase `registrations` table with `email`, `session_id`, `status_code`, `status_msg`, and `success` flag.
- Send Success Email1 — sends a success email to the user (requires SMTP credentials configured in n8n).
- Edit Fields1 / Insert Registration Record3 — on final failure, records the error payload and `finalRetryCount` and inserts a failing record into Supabase.

## Supabase schema (expected)

The workflow expects a table `registrations` with at least the following fields (names used in nodes):

- `email` (text)
- `session_id` (text)
- `status_code` (integer)
- `status_msg` (text)
- `success` (boolean)

Adjust field names in the Supabase nodes if your schema differs.

## Configuration / Credentials

1. Browserless
   - Open the `Register via Browserless1` HTTP Request node.
   - Replace the placeholder `YOUR_API_KEY` in the URL with your Browserless function token.
   - The node sends a `code` payload; review the embedded script to ensure the registration steps match your target application.

2. Supabase
   - Create and configure a Supabase credential in n8n (the workflow references a credential named "Supabase account").
   - Ensure the service role or API key used has permissions to INSERT into the `registrations` table.

3. SMTP (Email)
   - Configure an SMTP credential in n8n and attach it to the `Send Success Email1` node.
   - Update the `fromEmail` and subject/body templates as desired.

## Importing the workflow into n8n

1. In the n8n editor, click the hamburger menu (top-right) -> Import -> File and choose `Registration Automation (With Retry).json`.
2. Review and update node credentials (Supabase, SMTP) and any environment-specific URLs or tokens.
3. Activate the workflow when ready.

## Webhook path / endpoint

- The workflow's webhook node (`Webhook1`) exposes a path in n8n like `/webhook/ca5e6ac9-f558-4da7-80db-f88d434ea19b` (the exact base depends on your n8n host path). Use the n8n workflow editor to copy the full webhook URL.
- Send a POST request with JSON body { "email": "user@example.com", "password": "secret" } to trigger the workflow.

## Retry / Backoff behavior

- The retry condition triggers only when the Browserless response returns `statusCode` === 429.
- Retries are allowed while `retryCount < 5` (strictly less than 5). You can change the number in the `Retries Remaining?` If node.
- Wait node uses: amount = 5 * 2^retryCount (seconds). For example:
  - retryCount = 0 → 5s
  - retryCount = 1 → 10s
  - retryCount = 2 → 20s
  - retryCount = 3 → 40s

Adjust the multiplier or exponent base if you need faster/slower backoff.

## Testing tips

- Start with a test Browserless token and a disposable email to validate the happy path.
- To test retry logic, simulate a 429 response from Browserless or temporarily modify the `Check Status (429?)` node to check for a different code and return 429-like conditions.
- Inspect node executions in n8n to see fields passed between nodes (`sessionId`, `retryCount`, `statusCode`, and the Browserless response `body`).

## Troubleshooting

- If the workflow stops unexpectedly, check n8n system logs and node execution errors.
- If Supabase insert fails, confirm the credential and table permissions.
- If emails aren't sent, double-check SMTP settings and test sending from the SMTP provider directly.

## Customization ideas

- Record additional metadata (user agent, IP, request headers) in the registration record.
- Add notifying Slack/Teams on final failure or on repeated rate limiting.
- Add more granular error handling for non-429 errors (e.g., 500 server errors) with different retry strategies.

## Security

- Never commit API keys or secrets into the workflow JSON or this README. Use n8n credentials and environment variables.
- Ensure that the webhook endpoint is protected (e.g., verify a shared secret or accept requests only from trusted callers) if you expose it publicly.

## License

This repository does not include a license file. Add a LICENSE if you want to grant reuse permissions.

