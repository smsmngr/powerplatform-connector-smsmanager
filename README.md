# SmsManager — Power Platform Custom Connector

A Microsoft Power Platform custom connector for the [SmsManager](https://www.smsmanager.com)
SMS API. Use it to send SMS (and multi-channel) messages from Power Automate,
Power Apps, and Copilot Studio.

Full API documentation: <https://developers.smsmanager.com>

## What it does

The connector exposes one action:

| Action | Operation ID | Description |
|--------|--------------|-------------|
| **Send message** | `SendMessage` | Sends a message to one or more recipients (up to 10 per request). |

Under the hood it calls `POST https://api.smsmngr.com/v2/message` with the
`x-api-key` header for authentication.

### Request shape

```json
{
  "body": "Your verification code is 1234",
  "to": [
    { "phone_number": "+420777123456" }
  ],
  "tag": "transactional"
}
```

- `body` — the message text (required, max 1000 characters).
- `to` — array of recipients, each with a `phone_number` in international E.164
  format, **including** the leading `+` (for example `+420777123456`). 1–10 recipients.
- `tag` — optional label to group messages. Special values: `priority`, `transactional`.

### Delivery receipts

To receive delivery status updates, add a `callback` **object** to the request
body. You fully define how SmsManager POSTs each receipt — URL, HTTP method,
content type, headers and body template — so receipts arrive natively in exactly
the shape your target expects. There is no middleware and no fixed schema on
SmsManager's side.

`callback` object fields:

| Field | Type | Notes |
|-------|------|-------|
| `url` | string | **Required.** Where SmsManager sends the receipt. Must match `^https?://`, length 8–2048. |
| `method` | string | `POST` \| `PUT` \| `PATCH` \| `GET`. |
| `content_type` | string | `application/json` \| `application/x-www-form-urlencoded`. |
| `headers` | object | Custom headers your platform needs (auth tokens, signatures, …), up to 20 keys. |
| `item` | object \| array | Per-recipient template. |
| `body` | object \| array | Request body template SmsManager fills with receipt data. |
| `batch_size` | integer | 1–500. |
| `timeout_ms` | integer | 1000–10000. |

Because you supply `url` / `method` / `content_type` / `headers` / `body`,
SmsManager formats each outgoing receipt request in whatever shape the target
platform requires.

#### Power Platform example

Point the callback at a **Power Automate flow** that starts with the *"When a
HTTP request is received"* trigger. Copy that trigger's generated URL into
`callback.url`, then map SmsManager's receipt data onto the JSON fields your flow
expects:

```json
{
  "body": "Your verification code is 1234",
  "to": [
    { "phone_number": "+420777123456" }
  ],
  "tag": "transactional",
  "callback": {
    "url": "https://prod-00.westeurope.logic.azure.com/workflows/YOUR_WORKFLOW_ID/triggers/manual/paths/invoke?api-version=2016-06-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=YOUR_TRIGGER_SIGNATURE",
    "method": "POST",
    "content_type": "application/json",
    "headers": {
      "x-flow-token": "YOUR_SHARED_SECRET"
    },
    "body": {
      "messageId": "{{message_id}}",
      "recipient": "{{phone_number}}",
      "status": "{{status}}",
      "deliveredAt": "{{delivered_at}}"
    },
    "batch_size": 50,
    "timeout_ms": 5000
  }
}
```

Fill the placeholders (`YOUR_WORKFLOW_ID`, `YOUR_TRIGGER_SIGNATURE`,
`YOUR_SHARED_SECRET`) with the values from your own flow. Configure the flow
trigger's *Request Body JSON Schema* to match the `callback.body` template above
so the delivery-receipt fields land in named dynamic-content tokens.

> **Endpoint note.** The rich `callback` object shown here is delivered by the
> JSON API v2 path this connector uses
> (`POST https://api.smsmngr.com/v2/message`, JSON body, `x-api-key` header).
> Because the request is sent as JSON, the `callback` object can carry a fully
> custom receipt shape — URL, method, headers and body template — which is exactly
> what this connector does.

### Response shape

```json
{
  "request_id": "bc36f3d1-d284-463a-921b-a3560c154649",
  "accepted": [
    { "key": "0", "message_id": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" }
  ],
  "rejected": []
}
```

- `request_id` — unique identifier for the request.
- `accepted` — recipients accepted for delivery, each with a `key` (index in the
  original `to` array) and a `message_id`.
- `rejected` — recipients that were rejected, each with a `key`.

## API key setup

1. Sign in to your SmsManager account.
2. Open **Account settings → API** at <https://app.smsmanager.com/app/developers/apikeys>.
3. Create an API key.
4. When you create a connection to this connector in Power Platform, paste the key
   into the **API Key** field. It is sent as the `x-api-key` header on every request.

Never commit a real API key to source control. In examples above the demo recipient
number `+420777123456` is a placeholder.

## Files

| File | Purpose |
|------|---------|
| `apiDefinition.swagger.json` | Connector definition (OpenAPI / Swagger 2.0). |
| `apiProperties.json` | Connection parameters, branding, and policies. |
| `settings.json` | `paconn` settings (environment / connector placeholders). |
| `icon-placeholder.md` | Instructions for producing the required `icon.png`. |
| `LICENSE` | MIT license. |

> Power Platform requires **OpenAPI 2.0 (Swagger 2.0)**, not OpenAPI 3.0.

## Import

### Option A — `paconn` CLI

[`paconn`](https://learn.microsoft.com/connectors/custom-connectors/paconn-cli) is
Microsoft's Power Platform Connector CLI.

```bash
# Install
pip install paconn

# Authenticate
paconn login

# Fill in settings.json (environment + connector id), then create the connector
paconn create --api-def apiDefinition.swagger.json --api-prop apiProperties.json --icon icon.png

# To update an existing connector later
paconn update --settings settings.json
```

`paconn create` prints the new connector id — copy it into `settings.json` so that
future `paconn update` calls target the same connector.

### Option B — Power Platform portal

1. Go to <https://make.powerautomate.com> → **More → Discover all → Custom connectors**.
2. **New custom connector → Import an OpenAPI file** and select
   `apiDefinition.swagger.json`.
3. On the **General** tab set the icon (`icon.png`) and brand color `#A81943`.
4. On the **Security** tab confirm **API Key**, header name `x-api-key`.
5. **Create connector**, then test the **Send message** action with a real key.

## Certification

This repository is the connector source. Publishing it as a **certified** connector
that appears for all Power Platform customers is a **separate Microsoft submission
process** through the
[Power Platform Connector certification program](https://learn.microsoft.com/connectors/custom-connectors/submit-certification).
That process is independent of importing the connector as a custom (non-certified)
connector in your own environment, which works with the files here as-is.

## License

MIT — see [LICENSE](./LICENSE).
