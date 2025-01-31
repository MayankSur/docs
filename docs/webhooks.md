---
sidebar_position: 5
---

# Webhooks

You can configure the webhook URL in the [Heron dashboard](https://dashboard.herondata.io/) by navigating to the "Settings" tab on the left menu bar.

This is an example structure of our webhooks:

```json
{
    "topic": "end_user.processed",
    "created": "2021-05-20T09:23:53+00:00",
    "data": {
        "heron_id": "eus_Eqio3Y4dhyNiMphrXwG58p",
        "end_user_id": "myenduser",
        "status": "processed"
    },
    "meta": null
}
```

Where:

-   `topic` is the topic of this webhook in the format `<resource>.<event>`. Currently we support webhooks for the following topics:

    -   `end_user.processed`, triggered when asynchronous automated processing of an end user has finished. Processing is started after the EndUser status is set to "ready".
    -   `end_user.reviewed`, triggered when an underwriter / Heron has manually reviewed a company and set the EndUser status to "reviewed"
    -   `end_user.transactions_updated`, this webhook is in the process of being deprecated, please do not subscribe to this webhook topic.
    -   `end_user.review_required`, triggered when an end_user violates a rule during process and needs further review.
    -   `transactions.deleted`, triggered when transactions are deleted.
    -   `transactions.updated`, triggered within 10 minutes of the last change on transactions for a given end user (e.g. after feedback on category or Heron manual review).
    -   `pdf.processed`, triggered when Heron has successfully processed a PDF document.
    -   `pdf.transactions_loaded`, triggered when the transactions from a processed PDF document have been loaded into an end_user_id.
    -   `pdf.failed`, triggered when Heron has failed to process a PDF document.

-   `created` is the UTC datetime when the webhook was sent, in ISO format.
-   `data` contains the data of the resource which relates to this event.
-   `meta` (optional) contains further information about the event.

## Verification

We send a `Heron-Signature` header in every webhook request. This header is a
base64-encoded HMAC SHA256 digest of your shared secret and the webhook's
payload.

To verify the webhook was sent by us, calculate the digital signature using the
same algorithm and compare it to the `Heron-Signature` header.

Here is an example of how to calculate the signature in Python:

```py
import base64
import hashlib
import hmac
import json

secret = "sec_..." # shared secret, *not* your API credentials
data = {"topic": "end_user.processed", ...}

message = json.dumps(data, separators=(",", ":"))

dig = hmac.new(
    secret.encode("utf-8"),
    msg=message.encode("utf-8"),
    digestmod=hashlib.sha256,
).digest()

signature = base64.b64encode(dig).decode()
```

And in JavaScript (Node):

```js
const crypto = require('crypto')

const secret = 'sec_...'
const data = {"topic": "end_user.processed", ...}

const signature = crypto
  .createHmac('sha256', secret)
  .update(JSON.stringify(data))
  .digest('base64')
```

## Slack webhooks integration

We also support Slack [Incoming Webhooks](https://api.slack.com/messaging/webhooks) URLs when
you create a webhook. This allows you to fire alerts generated by Heron Data (e.g. a rule has been broken)
directly into a Slack channel for you to take action.

Follow these steps:

1. Follow the [Slack Incoming Webhook Documentation](https://api.slack.com/messaging/webhooks).
2. Slack will create a webhook url of the format `https://hooks.slack.com/services....` for you. Copy this url.
3. Open the Heron Data dashboard, navigate to the `Settings` page on the left, and go to `Webhooks`.
4. Press `Add webhook` and pick the appropriate topic. Most commonly, you'll want to fire a Slack alert when a rule has been broken. The webhook topic for this is `end_user.review_required`.
5. Press `Create webhook`.
6. To test the Slack webhook, re-process a company that breaks a rule. This will now fire a Slack alert!
