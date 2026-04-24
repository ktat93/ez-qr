<p align="center">
  <img src="assets/logo_white.svg" alt="EZQR" width="220" style="background:#1d2030;padding:16px 24px;border-radius:12px;" />
</p>

<h1 align="center">EZQR API</h1>

<p align="center">Generate, update, and track QR codes programmatically.</p> The EZQR API is a REST API that lets you create static and dynamic QR codes, manage landing pages, and pull scan analytics from your own systems.

Full documentation: https://ez-qr.com/docs/api
Live API reference: https://ez-qr.com/swagger

---

## What you can do with the API

- Create static or dynamic QR codes for any content type (URL, Wi-Fi, vCard, SMS, email, location, and 40+ more)
- Update a dynamic QR code's destination after it has already been printed
- Set up A/B tests, geo-targeting rules, and smart conditional routing on a single QR code
- Pull scan analytics broken down by date, device, country, browser, referrer, and UTM parameters
- Create and manage QR code landing pages with custom templates and branding
- Batch-create up to 100 QR codes in a single request

API access is available on the Max plan ($20/month). Rate limit is 100 requests per minute per API key.

---

## Base URL

```
https://ez-qr.com/api/v1
```

All requests must be made over HTTPS. HTTP requests will not be accepted.

---

## Authentication

Every request requires an API key passed as a Bearer token in the Authorization header.

```
Authorization: Bearer ezqr_your_api_key_here
```

API keys are generated from your EZQR dashboard under Settings > API. Keys start with `ezqr_` followed by a random string. Store your key securely -- it is shown only once on creation and cannot be retrieved again.

**401** is returned when the key is missing, malformed, or invalid.
**403** is returned when your plan does not include API access.
**429** is returned when you exceed 100 requests per minute.

---

## Quick start

Create your first QR code in under a minute:

```bash
curl -X POST https://ez-qr.com/api/v1/qr-codes \
  -H "Authorization: Bearer ezqr_your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Product page",
    "contentType": "url",
    "content": "https://yoursite.com/product",
    "type": "dynamic"
  }'
```

Response:

```json
{
  "data": {
    "id": "clx9f2g0000001",
    "name": "Product page",
    "type": "dynamic",
    "contentType": "url",
    "content": "https://yoursite.com/product",
    "shortCode": "abc123",
    "isActive": true,
    "totalScans": 0,
    "createdAt": "2026-02-15T10:00:00Z",
    "updatedAt": "2026-02-15T10:00:00Z"
  }
}
```

The `shortCode` field is the identifier used in the redirect URL: `https://ez-qr.com/r/abc123`. Print or embed that URL in your QR code image.

---

## Endpoints

### QR Codes

#### Create a QR code

```
POST /qr-codes
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | yes | Internal label for the code |
| contentType | string | yes | Type of content (see content types) |
| content | string | yes | The content value |
| type | string | no | "static" or "dynamic" (default: "static") |
| targetUrl | string | no | Redirect destination for dynamic codes |
| style | object | no | Visual customization settings |

A/B testing, geo-targeting, and smart rules are set after creation via `PATCH /qr-codes/:id`.

Static codes encode the content directly. Dynamic codes create a short redirect link that you can update without reprinting.

---

#### List QR codes

```
GET /qr-codes
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| limit | integer | 50 | Max results (max 100) |
| offset | integer | 0 | Pagination offset |
| type | string | none | Filter by "static" or "dynamic" |

Response includes `total`, `limit`, and `offset` fields alongside the `data` array.

---

#### Get a QR code

```
GET /qr-codes/:id
```

Returns a single QR code object by its ID. Returns 404 if the code does not exist or belongs to a different account.

---

#### Update a QR code

```
PATCH /qr-codes/:id
```

All fields are optional. Only the fields you send will be updated.

| Field | Type | Description |
|-------|------|-------------|
| name | string | Internal label |
| content | string | Content value |
| targetUrl | string or null | Redirect destination |
| isActive | boolean | Activate or deactivate the code |
| style | object or null | Visual styling |
| abTestConfig | object or null | A/B test settings |
| geoRules | object or null | Geo-targeting rules |
| smartRules | object or null | Conditional routing |
| folderId | string or null | Folder assignment |

Updating `targetUrl` on a dynamic code takes effect immediately for all future scans. No reprinting needed.

---

#### Delete a QR code

```
DELETE /qr-codes/:id
```

Returns `{ "success": true }` on success. This is permanent. Static codes are deleted immediately. Dynamic codes will stop redirecting after deletion.

---

#### Batch create QR codes

```
POST /qr-codes/batch
```

Create up to 100 QR codes in one request. Useful for product catalogs, location directories, or bulk campaign setup.

```json
{
  "codes": [
    {
      "name": "Location A",
      "contentType": "url",
      "content": "https://yoursite.com/location-a",
      "type": "dynamic"
    },
    {
      "name": "Location B",
      "contentType": "url",
      "content": "https://yoursite.com/location-b",
      "type": "dynamic"
    }
  ]
}
```

Response:

```json
{
  "data": {
    "created": 2,
    "errors": []
  }
}
```

Validation errors for individual items are returned in `errors` indexed by position. A failure on one item does not block the rest.

---

#### Get scan analytics

```
GET /qr-codes/:id/analytics?period=30d
```

| Parameter | Values | Default |
|-----------|--------|---------|
| period | 7d, 30d, 90d, all | 30d |

Response includes:

| Field | Description |
|-------|-------------|
| totalEvents | Total scan count for the period |
| uniqueVisitors | Unique scanners |
| eventsByDay | Daily scan counts |
| topDevices | Device breakdown |
| topBrowsers | Browser breakdown |
| topOs | Operating system breakdown |
| topCountries | Country breakdown |
| topCities | City breakdown |
| topReferrers | Referrer URLs |
| topSources | UTM source values |
| topMediums | UTM medium values |
| topCampaigns | UTM campaign names |

---

### Landing Pages

Landing pages (called showcases) are hosted pages at `https://ez-qr.com/s/your-slug`. You can link a landing page to any QR code and customize its appearance.

---

#### Create a landing page

```
POST /showcases
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| qrCodeId | string | yes | ID of the QR code to link |
| title | string | yes | Page heading |
| description | string | no | Subheading or body text |
| slug | string | no | Custom URL slug (auto-generated if omitted) |
| template | string | no | Visual template (default: "minimal") |

Available templates: `minimal`, `modern`, `bold`, `professional`, `dark`

After creation, use `PATCH /showcases/:id` to set colors, branding, and active status.

---

#### List landing pages

```
GET /showcases
```

Returns up to 50 landing pages on the account. No pagination parameters.

---

#### Get a landing page

```
GET /showcases/:id
```

---

#### Update a landing page

```
PATCH /showcases/:id
```

| Field | Type | Description |
|-------|------|-------------|
| title | string | Page heading |
| description | string or null | Subheading or body text |
| slug | string | Custom URL slug |
| template | string | Visual template |
| customColors | object or null | Override template colors |
| showBranding | boolean | Show EZQR badge |
| isActive | boolean | Activate or deactivate the page |

Returns 409 if the slug you provided is already taken by another page.

---

#### Delete a landing page

```
DELETE /showcases/:id
```

The page will immediately stop loading at its public URL.

---

## Advanced features

### A/B testing

Split traffic between two URLs on a single dynamic QR code. Useful for testing landing pages, offers, or destinations.

```json
{
  "abTestConfig": {
    "enabled": true,
    "variants": [
      { "name": "Version A", "url": "https://yoursite.com/a", "weight": 50 },
      { "name": "Version B", "url": "https://yoursite.com/b", "weight": 50 }
    ]
  }
}
```

Weights must add up to 100. Max plan supports unlimited variants. Pro plan supports 2 variants.

---

### Geo-targeting

Route scans to different URLs based on the scanner's country.

```json
{
  "geoRules": {
    "enabled": true,
    "defaultUrl": "https://yoursite.com/en",
    "rules": [
      { "country": "DE", "url": "https://yoursite.com/de" },
      { "country": "FR", "url": "https://yoursite.com/fr" }
    ]
  }
}
```

Country codes follow ISO 3166-1 alpha-2 format. Max plan supports unlimited rules. Pro plan supports 5 rules.

---

### Smart routing

Route scans based on conditions: time of day, day of week, date, device type, browser language, or scan count.

```json
{
  "smartRules": {
    "enabled": true,
    "defaultUrl": "https://yoursite.com",
    "rules": [
      {
        "id": "rule_1",
        "name": "Weekend deal",
        "priority": 1,
        "conditions": [
          { "type": "day", "value": ["saturday", "sunday"] }
        ],
        "url": "https://yoursite.com/weekend-offer",
        "enabled": true
      }
    ]
  }
}
```

Condition types: `time`, `day`, `date`, `device`, `language`, `scanCount`

Max plan supports unlimited rules (up to 20 per code). Pro plan supports 5 rules per code.

---

## Supported content types

| Content type | What it encodes |
|-------------|-----------------|
| url | Website URL |
| email | Email address with optional subject and body |
| phone | Phone number |
| sms | SMS message to a phone number |
| text | Plain text |
| wifi | Wi-Fi credentials (SSID, password, security type) |
| vcard | Contact card (name, phone, email, address, website) |
| whatsapp | WhatsApp message to a number |
| location | GPS coordinates or address |
| event | Calendar event (title, dates, location, description) |
| social | Social profile URL |
| app | App Store or Google Play link |

40+ specific subtypes are available under each category. The full list is at https://ez-qr.com/qr-codes.

---

## Error reference

| Code | Meaning |
|------|---------|
| 400 | Validation failed. Check the request body against the schema. |
| 401 | API key missing, malformed, or invalid. |
| 403 | Your plan does not support this feature or you have reached a plan limit. |
| 404 | Resource not found or does not belong to your account. |
| 409 | Slug conflict. Choose a different slug for the landing page. |
| 429 | Rate limit exceeded. You are allowed 100 requests per minute. |
| 500 | Server error. If this persists, contact support@ez-qr.com |

All errors return a JSON body with a `message` field explaining the failure.

---

## Rate limits

The API allows 100 requests per minute per API key. If you exceed this, you receive a 429 response. The limit resets on a 60-second rolling window.

If your use case requires higher throughput, contact support@ez-qr.com.

---

## Code examples

### Node.js

```javascript
const response = await fetch("https://ez-qr.com/api/v1/qr-codes", {
  method: "POST",
  headers: {
    "Authorization": "Bearer ezqr_your_key",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    name: "Product page",
    contentType: "url",
    content: "https://yoursite.com/product",
    type: "dynamic"
  })
});

const { data } = await response.json();
console.log(data.shortCode); // Use this in your QR image
```

### Python

```python
import requests

response = requests.post(
    "https://ez-qr.com/api/v1/qr-codes",
    headers={
        "Authorization": "Bearer ezqr_your_key",
        "Content-Type": "application/json"
    },
    json={
        "name": "Product page",
        "contentType": "url",
        "content": "https://yoursite.com/product",
        "type": "dynamic"
    }
)

data = response.json()["data"]
print(data["shortCode"])
```

### PHP

```php
$response = file_get_contents("https://ez-qr.com/api/v1/qr-codes", false, stream_context_create([
  "http" => [
    "method" => "POST",
    "header" => implode("\r\n", [
      "Authorization: Bearer ezqr_your_key",
      "Content-Type: application/json"
    ]),
    "content" => json_encode([
      "name" => "Product page",
      "contentType" => "url",
      "content" => "https://yoursite.com/product",
      "type" => "dynamic"
    ])
  ]
]));

$data = json_decode($response, true)["data"];
echo $data["shortCode"];
```

---

## Frequently asked questions

**Do I need an account to use the API?**
Yes. API access requires an EZQR account on the Max plan. You can sign up at https://ez-qr.com.

**What happens to my QR codes if I cancel?**
Static codes are yours and do not depend on your subscription. Dynamic codes stay active for 30 days after cancellation. There is no annual billing -- subscriptions are month to month.

**Can I generate the QR code image itself through the API?**
The API returns a short redirect URL (`ez-qr.com/r/shortCode`) that you encode into a QR image using any QR generation library. The EZQR dashboard generates styled PNG, SVG, and PDF versions for download.

**What is the difference between a static and dynamic QR code?**
A static code encodes the content directly -- changing it means reprinting. A dynamic code points to a redirect URL that you control. You update the destination through the dashboard or API and the same printed code still works.

**Can I update a QR code's destination after printing?**
Yes, if it is a dynamic code. Send a PATCH request to `/qr-codes/:id` with the new `targetUrl` and scans will redirect to the new destination immediately.

**Is there a sandbox environment?**
Not currently. Test against the live API with a real account.

**Where do I get my API key?**
Log in to EZQR, go to Settings > API, and create a new key. Keys are only shown once on creation.

---

## OpenAPI spec

A machine-readable OpenAPI 3.1.0 specification is available at:

```
GET https://ez-qr.com/.well-known/openapi
```

You can import this into Postman, Insomnia, or any API client that supports OpenAPI.

An interactive Swagger UI is available at https://ez-qr.com/swagger.

---

## Support

For API-specific issues: support@ez-qr.com
For general questions: https://ez-qr.com/faq
For bugs or feature requests: open an issue in this repository

---

*EZQR is a QR code generator with dynamic codes, scan analytics, and no annual billing. Plans start at $5/month. https://ez-qr.com*
