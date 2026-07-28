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
    { "phone_number": "420777123456" }
  ],
  "tag": "transactional"
}
```

- `body` — the message text (required, max 1000 characters).
- `to` — array of recipients, each with a `phone_number` in international E.164
  format **without** a leading `+` or `00` (for example `420777123456`). 1–10 recipients.
- `tag` — optional label to group messages. Special values: `priority`, `transactional`.

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
2. Open **Account settings → API** at <https://app.smsmanager.com/api-cloud>.
3. Create an API key.
4. When you create a connection to this connector in Power Platform, paste the key
   into the **API Key** field. It is sent as the `x-api-key` header on every request.

Never commit a real API key to source control. In examples above the demo recipient
number `420777123456` is a placeholder.

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
