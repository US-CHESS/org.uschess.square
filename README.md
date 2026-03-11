# org.uschess.square

Square payment processor extension for [CiviCRM](https://civicrm.org/), developed by the [US Chess Federation](https://www.uschess.org).

## Features

- **One-time payments** via the Square Payments API (`/v2/payments`)
- **Recurring contributions** via Square Subscriptions API (plans, plan variations, customer cards)
- **PCI-compliant tokenization** using the [Square Web Payments SDK](https://developer.squareup.com/docs/web-payments/overview) -- card details never touch your server
- **Square Customer sync** -- creates/links Square Customers to CiviCRM contacts; stores `square_customer_id` and `square_card_id` as custom fields
- **Webhook support** for payment and subscription lifecycle events (`payment.completed`, `subscription.created`, `subscription.updated`, `subscription.canceled`, `invoice.payment_made`)
- **Refunds** via the CiviCRM admin UI
- **Sandbox/test mode** with per-environment credentials

## Compatibility

| Component | Version |
|---|---|
| CiviCRM | 5.69+ |
| PHP | 8.1+ |
| Drupal | 10 / 11 |
| Square PHP SDK | ^43.2 |

Works with:

- CiviCRM Contribution Pages
- CiviCRM Event Registration
- Drupal Webform + Webform CiviCRM Integration

## Installation

### Via Composer (recommended)

```bash
composer require uschess/org.uschess.square
cv ext:enable org.uschess.square
```

### Manual

1. Download or clone this repository into your CiviCRM extensions directory (typically `sites/default/files/civicrm/ext/`).
2. Run `composer install` inside the extension directory to install the Square PHP SDK.
3. Enable the extension: **Administer > System Settings > Extensions**.

## Configuration

1. Go to **Administer > System Settings > Payment Processors**.
2. Click **Add Payment Processor** and select **Square**.
3. Fill in credentials from your [Square Developer Dashboard](https://developer.squareup.com/apps):

| Field | Description |
|---|---|
| Square Application ID | From Credentials tab |
| Square Access Token | Production or sandbox access token |
| Square Location ID | From Locations in your Square Dashboard |
| Webhook Signature Key | From Webhooks settings (used to verify incoming events) |

4. For test/sandbox mode, fill in the corresponding sandbox fields and checkout using TEST mode on any form.

## Webhook Setup

1. In the Square Developer Dashboard, add a webhook subscription pointing to:
   ```
   https://your-site.example.com/civicrm/square/webhook
   ```
2. Subscribe to events: `payment.completed`, `invoice.payment_made`, `subscription.created`, `subscription.updated`, `subscription.canceled`.
3. Copy the **Signature Key** into the payment processor configuration.

## How It Works

1. The extension registers a `Square` payment processor type via a managed entity.
2. On contribution/event forms, it injects the Square Web Payments SDK and a card entry iframe.
3. On form submission, the SDK tokenizes the card client-side and puts the nonce into a hidden field (`square_payment_token`).
4. `CRM_Core_Payment_Square::doPayment()` sends the token to the Square Payments API and records the transaction in CiviCRM.
5. For recurring contributions, a Square Subscription is created and linked to the CiviCRM recurring contribution record. Webhooks keep the records in sync.

## Webform CiviCRM Compatibility

This extension includes specific support for Drupal Webform CiviCRM:

- `js/square.js` dynamically loads the Square SDK and re-initializes after Webform's AJAX billing block injection.
- The submit handler uses capture-phase interception to tokenize before other handlers run.
- Token is passed to `doPayment()` via both the standard params array and a `$_POST` fallback for the confirm-form code path.

> **Note:** Webform CiviCRM's confirm-form payment path (`WebformCivicrmConfirmForm::doPayment()`) does not merge `$_POST` values into payment params by default. A fix has been submitted upstream: [colemanw/webform_civicrm#1111](https://github.com/colemanw/webform_civicrm/pull/1111). This extension includes a server-side `$_POST` fallback so it works with or without the upstream fix, but merging that PR is recommended for any tokenized payment processor.


## License

This extension is licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html).

## Credits

Developed and maintained by the [US Chess Federation](https://new.uschess.org).
