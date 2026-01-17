---
name: visiono-api
description: Use this skill when the user asks about Visiono API integration, Photo Requests, Smart Links, Workflow Mode, webhooks, or photo collection workflows. Provides API documentation, code examples in PHP/JavaScript/Python, webhook signature verification, and Workflow Mode signed URL generation patterns.
version: 1.1.0
---

# Visiono API Integration Specialist

## CRITICAL: Always Fetch Latest Docs First

Before ANY implementation, fetch current OpenAPI spec:
```
curl -s https://www.visiono.io/docs/api/v1.openapi
```
Or use WebFetch on `https://www.visiono.io/docs/api/v1` - docs may change.

## Quick Reference

### Authentication
```
Header: X-API-Key: vsk_live_<your_key>
```

### Base URL
```
https://www.visiono.io/api/v1
```

### ID Prefixes
| Prefix | Resource |
|--------|----------|
| `pr_` | Photo Request |
| `pri_` | Photo Request Item |
| `ps_` | Photo Submission |
| `sl_` | Smart Link |
| `pls_` | Permanent Link Submission |
| `wh_` | Webhook Endpoint |

### Rate Limiting
Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`

---

## Photo Requests API

### Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/photo-requests` | List (filters: status, created_after, created_before) |
| POST | `/photo-requests` | Create |
| POST | `/photo-requests/batch` | Batch create (max 10) |
| GET | `/photo-requests/{id}` | Get by ID/code |
| DELETE | `/photo-requests/{id}` | Cancel |
| GET | `/photo-requests/{id}/submissions` | Get submissions |
| GET | `/photo-requests/{id}/download-urls` | Get signed URLs (24h valid) |
| GET | `/photo-requests/{id}/qrcode` | Get QR code URL |

### POST /photo-requests - Create

**Request:**
```json
{
  "instructions": "Upload your ID",
  "recipient_email": "user@example.com",
  "expires_in_hours": 48,
  "require_gps": false,
  "allow_submitter_notes": true,
  "send_email": true,
  "internal_notes": "Ticket #123",
  "metadata": {"ticket_id": "12345"},
  "items": [
    {"instructions": "Front of ID", "is_required": true},
    {"instructions": "Back of ID", "is_required": true}
  ]
}
```

**Response (201):**
```json
{
  "data": {
    "id": "pr_1",
    "unique_code": "ABC123",
    "numeric_code": "809530",
    "status": "pending",
    "instructions": "Upload your ID",
    "recipient_email": "user@example.com",
    "request_url": "https://tenant.visio.now/ABC123",
    "code_entry_url": "https://visio.now/code",
    "expires_at": "2024-12-12T12:00:00+00:00",
    "created_at": "2024-12-10T12:00:00+00:00",
    "metadata": {"ticket_id": "12345"},
    "items": [
      {"id": "pri_1", "instructions": "Front of ID", "is_required": true},
      {"id": "pri_2", "instructions": "Back of ID", "is_required": true}
    ]
  }
}
```

### GET /photo-requests/{id}/qrcode - Get QR Code

> 💡 **No external QR libraries needed!** Use this URL directly in `<img src="">`.

**Response (200):**
```json
{
  "data": {
    "qrcode_url": "https://visio.now/qr/ABC123.png"
  }
}
```

**Usage:**
```html
<img src="${response.data.qrcode_url}" alt="Scan to upload photos" />
```

```javascript
// Fetch QR code URL
const res = await fetch(`/api/v1/photo-requests/${id}/qrcode`, {
  headers: { 'X-API-Key': apiKey }
});
const { data } = await res.json();
document.getElementById('qr').src = data.qrcode_url;
```

### GET /photo-requests/{id}/download-urls - Get Photo Downloads

**Response (200):**
```json
{
  "data": {
    "photo_request_id": "pr_1",
    "submissions": [
      {
        "id": "ps_1",
        "item_id": "pri_1",
        "download_url": "https://storage.../photo1.jpg?signature=...",
        "thumbnail_url": "https://storage.../photo1_thumb.jpg?signature=...",
        "file_size_kb": 1024,
        "mime_type": "image/jpeg",
        "has_gps_data": true,
        "gps_latitude": 45.4642,
        "gps_longitude": 9.1900,
        "submitted_at": "2024-12-11T10:30:00+00:00"
      },
      {
        "id": "ps_2",
        "item_id": "pri_2",
        "download_url": "https://storage.../photo2.jpg?signature=...",
        "thumbnail_url": "https://storage.../photo2_thumb.jpg?signature=...",
        "file_size_kb": 856,
        "mime_type": "image/jpeg",
        "has_gps_data": false,
        "submitted_at": "2024-12-11T10:31:00+00:00"
      }
    ]
  },
  "meta": {
    "urls_valid_for": "24 hours",
    "expires_in_seconds": 86400
  }
}
```

**Usage:**
```javascript
// Download all photos
const { data } = await response.json();
for (const sub of data.submissions) {
  console.log(`Photo ${sub.id}:`);
  console.log(`  Download: ${sub.download_url}`);
  console.log(`  Thumbnail: ${sub.thumbnail_url}`);
  if (sub.has_gps_data) {
    console.log(`  GPS: ${sub.gps_latitude}, ${sub.gps_longitude}`);
  }
}
```

---

## Smart Links API

### Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/smart-links` | List (filters: is_active, tags) |
| POST | `/smart-links` | Create |
| GET | `/smart-links/{id}` | Get |
| PUT | `/smart-links/{id}` | Update |
| DELETE | `/smart-links/{id}` | Archive |
| GET | `/smart-links/{id}/submissions` | Get submissions |
| GET | `/smart-links/{id}/download-urls` | Get signed URLs |
| GET | `/smart-links/{id}/qrcode` | Get QR code URL |

### POST /smart-links - Create

**Request:**
```json
{
  "instructions": "Vehicle inspection photos",
  "custom_slug": "fleet-inspection",
  "unique_field_label": "License Plate",
  "show_full_name_field": true,
  "full_name_required": true,
  "require_gps": true,
  "tags": ["fleet", "inspection"],
  "items": [
    {"instructions": "Front view", "is_required": true},
    {"instructions": "Damage (if any)", "is_required": false}
  ]
}
```

**Response (201):**
```json
{
  "data": {
    "id": "sl_1",
    "unique_code": "XYZ789",
    "custom_slug": "fleet-inspection",
    "is_active": true,
    "instructions": "Vehicle inspection photos",
    "unique_field_label": "License Plate",
    "request_url": "https://tenant.visio.now/fleet-inspection",
    "created_at": "2024-12-10T12:00:00+00:00",
    "tags": ["fleet", "inspection"],
    "items": [
      {"id": "pri_10", "instructions": "Front view", "is_required": true},
      {"id": "pri_11", "instructions": "Damage (if any)", "is_required": false}
    ]
  }
}
```

### GET /smart-links/{id}/qrcode - Get QR Code

**Response (200):**
```json
{
  "data": {
    "qrcode_url": "https://visio.now/qr/fleet-inspection.png"
  }
}
```

---

## Workflow Mode

Workflow Mode allows external systems to embed Visiono photo collection into their processes using **HMAC-signed URLs**. When enabled on a Smart Link, direct access is blocked - all access requires signed URLs.

> ⚠️ **Important**: When Workflow Mode is enabled, the link can NO LONGER be shared directly. Users opening the link without valid signed parameters will see an error page.

### How It Works

1. **Your system** generates a signed URL with parameters
2. **User** opens the signed URL
3. **Visiono** validates signature and shows upload interface
4. **User** uploads photos
5. **Visiono** redirects back to your callback URL with signed data
6. **Your system** verifies the return signature

### Input URL Parameters (to Visiono)

| Param | Description | Required |
|-------|-------------|----------|
| `uv` | Unique value (e.g., order ID, ticket number) | Yes |
| `src` | Source system identifier (e.g., "zendesk", "crm") | Yes |
| `ref` | External reference (e.g., ticket-456) | Yes |
| `ts` | Unix timestamp (seconds) | Yes |
| `sig` | HMAC-SHA256 signature | Yes |

### Input Signature Algorithm

```
payload = "{uv}|{src}|{ref}|{ts}"
signature = HMAC-SHA256(payload, workflow_secret)
```

### Generate Signed URL - PHP

```php
$workflowSecret = 'your-64-char-workflow-secret';
$slug = 'damage-report';

$uniqueValue = 'ABC123';
$source = 'zendesk';
$reference = 'ticket-456';
$timestamp = time();

// Generate signature
$payload = "{$uniqueValue}|{$source}|{$reference}|{$timestamp}";
$signature = hash_hmac('sha256', $payload, $workflowSecret);

// Build URL
$baseUrl = "https://visiono.io/s/{$slug}";
$params = http_build_query([
    'uv' => $uniqueValue,
    'src' => $source,
    'ref' => $reference,
    'ts' => $timestamp,
    'sig' => $signature,
]);

$url = "{$baseUrl}?{$params}";
// https://visiono.io/s/damage-report?uv=ABC123&src=zendesk&ref=ticket-456&ts=1705500000&sig=abc123...
```

### Generate Signed URL - JavaScript

```javascript
const crypto = require('crypto');

const workflowSecret = 'your-64-char-workflow-secret';
const slug = 'damage-report';

const uniqueValue = 'ABC123';
const source = 'zendesk';
const reference = 'ticket-456';
const timestamp = Math.floor(Date.now() / 1000);

// Generate signature
const payload = `${uniqueValue}|${source}|${reference}|${timestamp}`;
const signature = crypto
  .createHmac('sha256', workflowSecret)
  .update(payload)
  .digest('hex');

// Build URL
const params = new URLSearchParams({
  uv: uniqueValue,
  src: source,
  ref: reference,
  ts: timestamp,
  sig: signature,
});

const url = `https://visiono.io/s/${slug}?${params}`;
```

### Generate Signed URL - Python

```python
import hmac
import hashlib
import time
from urllib.parse import urlencode

workflow_secret = 'your-64-char-workflow-secret'
slug = 'damage-report'

unique_value = 'ABC123'
source = 'zendesk'
reference = 'ticket-456'
timestamp = int(time.time())

# Generate signature
payload = f"{unique_value}|{source}|{reference}|{timestamp}"
signature = hmac.new(
    workflow_secret.encode(),
    payload.encode(),
    hashlib.sha256
).hexdigest()

# Build URL
params = urlencode({
    'uv': unique_value,
    'src': source,
    'ref': reference,
    'ts': timestamp,
    'sig': signature,
})

url = f"https://visiono.io/s/{slug}?{params}"
```

### Output URL Parameters (from Visiono redirect)

When the user completes the upload, Visiono redirects to your configured callback URL with these parameters:

| Param | Description |
|-------|-------------|
| `sub` | Submission ID (e.g., `pls_123`) |
| `uv` | Unique value (echoed back) |
| `ref` | Reference (echoed back) |
| `cnt` | Number of photos uploaded |
| `ts` | Timestamp of completion |
| `sig` | Return signature for verification |

### Return Signature Algorithm

```
payload = "{submission_id}|{unique_value}|{reference}|{photo_count}|{timestamp}"
signature = HMAC-SHA256(payload, workflow_secret)
```

### Verify Return Signature - PHP

```php
$workflowSecret = 'your-64-char-workflow-secret';

// Get parameters from redirect
$submissionId = $_GET['sub'];
$uniqueValue = $_GET['uv'];
$reference = $_GET['ref'];
$photoCount = $_GET['cnt'];
$timestamp = $_GET['ts'];
$receivedSignature = $_GET['sig'];

// Verify signature
$payload = "{$submissionId}|{$uniqueValue}|{$reference}|{$photoCount}|{$timestamp}";
$expectedSignature = hash_hmac('sha256', $payload, $workflowSecret);

if (hash_equals($expectedSignature, $receivedSignature)) {
    // ✅ Signature valid - process the submission
    echo "Photos submitted successfully!";
} else {
    // ❌ Signature invalid - reject
    http_response_code(400);
    echo "Invalid signature";
}
```

### Verify Return Signature - JavaScript

```javascript
const crypto = require('crypto');

function verifyRedirectSignature(params, workflowSecret) {
  const { sub, uv, ref, cnt, ts, sig } = params;

  const payload = `${sub}|${uv}|${ref}|${cnt}|${ts}`;
  const expected = crypto
    .createHmac('sha256', workflowSecret)
    .update(payload)
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(sig),
    Buffer.from(expected)
  );
}
```

### Verify Return Signature - Python

```python
import hmac
import hashlib

def verify_redirect_signature(params, workflow_secret):
    payload = f"{params['sub']}|{params['uv']}|{params['ref']}|{params['cnt']}|{params['ts']}"
    expected = hmac.new(
        workflow_secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(expected, params['sig'])
```

### Link Expiration

Signed URLs expire after **1 hour** by default. This prevents:
- Old links from being reused
- Replay attacks
- Stale data

### Error Pages

Users may see error pages when:

| Error | Cause | Solution |
|-------|-------|----------|
| Missing Parameters | URL lacks required params | Check URL generation |
| Invalid Signature | Signature doesn't match | Verify secret and algorithm |
| Link Expired | Timestamp > 1 hour old | Generate new signed URL |

### Workflow Mode + Webhooks

When using Workflow Mode, the `submission.created` webhook payload includes external tracking fields:

```json
{
  "event": "submission.created",
  "data": {
    "photo_request": {
      "id": "sl_123",
      "custom_slug": "damage-report"
    },
    "permanent_link_submission": {
      "id": "pls_456",
      "unique_field_value": "ABC123",
      "external_source": "zendesk",
      "external_reference": "ticket-456"
    }
  }
}
```

### Workflow Mode Best Practices

1. **Store secret securely** - Use environment variables, never client-side code
2. **Verify all signatures** - Always validate return signatures
3. **Use HTTPS** - All URLs should be secure
4. **Regenerate secrets** - If compromised, regenerate in Visiono dashboard
5. **Handle expiration** - Generate fresh URLs for each request

---

## Webhooks API

### Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/webhooks` | List endpoints |
| POST | `/webhooks` | Create (returns secret once!) |
| GET | `/webhooks/{id}` | Get |
| PATCH | `/webhooks/{id}` | Update |
| DELETE | `/webhooks/{id}` | Delete |
| POST | `/webhooks/{id}/test` | Send test |
| GET | `/webhooks/events` | List available events |

### POST /webhooks - Create

**Request:**
```json
{
  "url": "https://your-server.com/webhook",
  "events": ["photo_submission.created", "photo_request.submitted"]
}
```

**Response (201):** ⚠️ **Secret shown only once!**
```json
{
  "data": {
    "id": "wh_1",
    "url": "https://your-server.com/webhook",
    "events": ["photo_submission.created", "photo_request.submitted"],
    "secret": "whsec_abc123xyz789...",
    "is_active": true,
    "created_at": "2024-12-10T12:00:00+00:00"
  }
}
```

### Webhook Events

| Event | Trigger |
|-------|---------|
| `photo_request.created` | New request created |
| `photo_request.first_viewed` | Recipient opened the link |
| `photo_submission.created` | Single photo uploaded |
| `photo_request.submitted` | All required photos done |
| `photo_request.expired` | Request expired without completion |

### Webhook Headers

| Header | Description |
|--------|-------------|
| `X-Visiono-Event` | Event type (e.g., `photo_submission.created`) |
| `X-Visiono-Delivery` | Unique delivery ID (for deduplication) |
| `X-Visiono-Signature` | HMAC-SHA256 signature |
| `X-Visiono-Tenant-Id` | Your tenant ID |

### Webhook Payload Examples

**photo_request.created:**
```json
{
  "event": "photo_request.created",
  "timestamp": "2024-12-25T12:00:00Z",
  "data": {
    "photo_request": {
      "id": "pr_123",
      "unique_code": "ABC123",
      "status": "pending",
      "recipient_email": "user@example.com",
      "request_url": "https://tenant.visio.now/ABC123",
      "expires_at": "2024-12-27T12:00:00Z",
      "metadata": {"ticket_id": "12345"}
    }
  }
}
```

**photo_submission.created:**
```json
{
  "event": "photo_submission.created",
  "timestamp": "2024-12-25T14:30:00Z",
  "data": {
    "photo_request": {
      "id": "pr_123",
      "unique_code": "ABC123",
      "status": "in_progress",
      "metadata": {"ticket_id": "12345"}
    },
    "submission": {
      "id": "ps_456",
      "item_id": "pri_1",
      "file_size_kb": 1024,
      "mime_type": "image/jpeg",
      "has_gps_data": true,
      "gps_latitude": 45.4642,
      "gps_longitude": 9.1900,
      "submitted_at": "2024-12-25T14:30:00Z"
    }
  }
}
```

**photo_request.submitted:**
```json
{
  "event": "photo_request.submitted",
  "timestamp": "2024-12-25T14:35:00Z",
  "data": {
    "photo_request": {
      "id": "pr_123",
      "unique_code": "ABC123",
      "status": "submitted",
      "submissions_count": 2,
      "metadata": {"ticket_id": "12345"},
      "submitted_at": "2024-12-25T14:35:00Z"
    }
  }
}
```

### Signature Verification (CRITICAL)

Format: `sha256=<hmac_hex>`

**PHP:**
```php
$payload = file_get_contents('php://input');
$expected = 'sha256=' . hash_hmac('sha256', $payload, $secret);
$valid = hash_equals($expected, $_SERVER['HTTP_X_VISIONO_SIGNATURE']);
```

**JavaScript:**
```javascript
const crypto = require('crypto');
const expected = 'sha256=' + crypto.createHmac('sha256', secret).update(payload).digest('hex');
const valid = crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected));
```

**Python:**
```python
import hmac, hashlib
expected = 'sha256=' + hmac.new(secret.encode(), payload, hashlib.sha256).hexdigest()
valid = hmac.compare_digest(expected, signature)
```

### Retry Policy
- 3 attempts with backoff: 1min, 5min, 30min
- Timeout: 30s
- Success: HTTP 2xx

---

## API Scopes

| Scope | Permission |
|-------|------------|
| `photo_requests:read` | List/view requests |
| `photo_requests:write` | Create/cancel requests |
| `submissions:read` | View submissions |
| `download_urls:read` | Get download URLs |
| `smart_links:read` | List/view smart links |
| `smart_links:write` | Create/modify smart links |
| `webhooks:read` | List webhooks |
| `webhooks:write` | Manage webhooks |

## Common Errors

| Code | Error | Solution |
|------|-------|----------|
| 401 | Unauthenticated | Check X-API-Key header |
| 403 | Missing scope | Add required scope to API key |
| 404 | Not found | Verify resource ID |
| 422 | Validation error | Check required fields |
| 429 | Rate limited | Implement backoff |

## Use Cases

| Use Case | Type | Key Params |
|----------|------|------------|
| ID verification | Photo Request | expires_in_hours: 24, items: front/back/selfie |
| Warranty claim | Photo Request | items: product, defect, receipt |
| Fleet inspection | Smart Link | unique_field_label: "Plate", require_gps: true |
| Property check-in | Smart Link | require_gps: true, show_full_name_field: true |

---

## QR Code - Don't Generate, Just Use!

### ❌ Don't Do This
```javascript
// WRONG: Don't use external QR libraries
import QRCode from 'qrcode';
const qr = await QRCode.toDataURL(requestUrl);  // ← UNNECESSARY!
```

### ✅ Do This Instead
```javascript
// CORRECT: Use Visiono's ready-made QR code
const res = await fetch(`/api/v1/photo-requests/${id}/qrcode`, {
  headers: { 'X-API-Key': apiKey }
});
const { data } = await res.json();
// data.qrcode_url = "https://visio.now/qr/ABC123.png"

document.getElementById('qr').src = data.qrcode_url;
```

```html
<!-- Just embed the URL directly -->
<img src="${data.qrcode_url}" alt="Scan to upload photos" width="200" height="200" />
```

---

## Polling Pattern for Webhooks

When polling for webhook events (instead of real-time webhooks), **ALWAYS** filter by:
1. Request ID - only events for YOUR request
2. Event timestamp - only events AFTER request creation

```javascript
// ✅ CORRECT: Filter by ID AND timestamp
function startPolling(requestId, requestCreatedAt) {
  const createdTime = new Date(requestCreatedAt).getTime();

  const interval = setInterval(async () => {
    const events = await fetch('/your-webhook-storage').then(r => r.json());

    const newEvent = events.find(e => {
      // Only submission events
      if (!['photo_submission.created', 'photo_request.submitted'].includes(e.event)) {
        return false;
      }

      // Only events AFTER request was created
      const eventTime = new Date(e.timestamp).getTime();
      if (eventTime < createdTime) return false;

      // Only for THIS request
      return e.data.photo_request?.id === requestId;
    });

    if (newEvent) {
      clearInterval(interval);
      handleSubmission(newEvent);
    }
  }, 2000);
}
```

### ❌ Common Mistake
```javascript
// WRONG: No timestamp filter - will match OLD events!
const event = events.find(e => e.data.photo_request?.id === requestId);
```

> ⚠️ **Without timestamp filtering, you'll find events from previous requests with the same ID pattern!**

---

## Webhook Event Storage Best Practices

When storing webhook events (e.g., in KV, Redis, or database):

### ✅ DO: Store by Request ID
```javascript
// Separate storage per request
await kv.put(`events:${requestId}`, JSON.stringify(events));

// Easy to retrieve and clean up
const events = await kv.get(`events:${requestId}`);
```

### ❌ DON'T: Global Event Storage
```javascript
// Single array for all events - causes conflicts!
await kv.put('all_events', JSON.stringify(allEvents));
```

### Storage Guidelines

| Rule | Reason |
|------|--------|
| **Namespace by request ID** | Prevents cross-request pollution |
| **Add received timestamp** | For filtering and debugging |
| **Limit to ~100 events** | Prevent storage bloat |
| **TTL of 24 hours** | Old events are useless |
| **Store delivery ID** | Detect and skip duplicates |

```javascript
// Example: Cloudflare Worker KV storage
async function storeWebhookEvent(event) {
  const requestId = event.data.photo_request?.id;
  const key = `webhook:${requestId}`;

  let events = JSON.parse(await KV.get(key) || '[]');

  // Skip duplicates
  if (events.some(e => e.delivery_id === event.delivery_id)) {
    return;
  }

  events.push({
    ...event,
    received_at: new Date().toISOString()
  });

  // Keep only last 50 events
  events = events.slice(-50);

  // Store with 24h TTL
  await KV.put(key, JSON.stringify(events), { expirationTtl: 86400 });
}
```

---

## Implementation Checklist

1. Store API key in env vars (never in code)
2. Always verify webhook signatures
3. Handle rate limits with exponential backoff
4. Download photos promptly (URLs expire in 24h)
5. Use batch endpoint for multiple requests
6. Store webhook delivery IDs to detect duplicates
7. Use HTTPS for webhook endpoints
8. **Use Visiono's QR codes - don't generate your own!**

## Integration Guides Available

- Workflow Mode: `/docs/integrations/workflow-mode` ⭐ NEW
- Zapier: `/docs/integrations/zapier`
- Make: `/docs/integrations/make`
- n8n: `/docs/integrations/n8n`
- Zendesk: `/docs/integrations/zendesk`
- Monday.com: `/docs/integrations/monday`
- Notion: `/docs/integrations/notion`
- Claude Code: `/docs/integrations/claude-code`
