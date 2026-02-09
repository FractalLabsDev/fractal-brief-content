Vitaboom Vendor API Documentation

# Vitaboom Vendor API Documentation

> **Comprehensive API reference for vendor integrations with Vitaboom's Shopify order management system**

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Base URL](#base-url)
- [API Endpoints](#api-endpoints)
  - [1. Health Check](#1-health-check)
  - [2. Create Order](#2-create-order)
  - [3. Retrieve Order](#3-retrieve-order)
  - [4. List Orders](#4-list-orders)
  - [5. Get Order Fulfillments](#5-get-order-fulfillments)
  - [6. Update Order](#6-update-order)
  - [7. Cancel Order](#7-cancel-order)
- [Webhooks](#webhooks)
  - [Setup](#setup)
  - [Webhook Event Types](#webhook-event-types)
  - [Webhook Payload Structure](#webhook-payload-structure)
  - [Webhook Field Details](#webhook-field-details)
  - [Webhook Headers](#webhook-headers)
  - [Event-Specific Payloads](#event-specific-payloads)
  - [Webhook Retry Logic](#webhook-retry-logic)
  - [Webhook Best Practices](#webhook-best-practices)
- [Rate Limits](#rate-limits)
- [Error Handling](#error-handling)
  - [Error Response Format](#error-response-format)
  - [Common Error Codes](#common-error-codes)
  - [Retry Guidelines](#retry-guidelines)
- [Data Formats](#data-formats)
  - [Date/Time](#datetime)
  - [Shopify IDs](#shopify-ids)
  - [Country Codes](#country-codes)
  - [Province/State Codes](#provincestate-codes)
  - [Currency Codes](#currency-codes)
  - [Currency Values](#currency-values)
- [Testing](#testing)
  - [Test Mode](#test-mode)
  - [Sample Test Data](#sample-test-data)
- [Support & Contact](#support--contact)
- [Changelog](#changelog)
- [Appendix: Complete API Reference](#appendix-complete-api-reference)
  - [All Available Endpoints](#all-available-endpoints)
  - [All Webhook Events](#all-webhook-events)

---

## Overview

This documentation provides a complete reference for all API endpoints and webhooks available to vendor partners integrating with Vitaboom's order fulfillment system. Each vendor receives a unique `vendorSlug` identifier used in API routes and automatic order tagging.

---

## Authentication

All API requests require authentication via API key header:

```
X-API-Key: <your-api-key>
```

**Security Best Practices:**
- Never expose API keys in client-side code
- Store credentials securely (environment variables, secrets management)
- Never commit API keys to version control
- Rotate keys periodically

---

## Base URL

```
https://vitaboom-backend-prod-960e6ca03e76.herokuapp.com
```

---

## API Endpoints

### 1. Health Check

**Endpoint:** `GET /api/health`

**Description:** Verify API availability and connectivity

**Authentication:** Not required

**Response:**
```json
{
  "status": "ok"
}
```

**Status Codes:**
- `200 OK` - Service operational

---

### 2. Create Order

**Endpoint:** `POST /api/vendor/{vendorSlug}/orders`

**Description:** Create a new order in Shopify through Vitaboom's order management system. Orders are automatically tagged with the vendor's name for tracking and reporting.

**Authentication:** Required

**Path Parameters:**
- `vendorSlug` (string, required) - Your unique vendor identifier

**Request Body:**

```json
{
  "sourceIdentifier": "string",
  "referringSite": "string",
  "email": "string",
  "phone": "string",
  "financialStatus": "string",
  "currency": "string",
  "lineItems": [
    {
      "variantId": "string",
      "quantity": number,
      "requiresShipping": boolean
    }
  ],
  "shippingAddress": {
    "firstName": "string",
    "lastName": "string",
    "company": "string",
    "address1": "string",
    "address2": "string",
    "city": "string",
    "provinceCode": "string",
    "countryCode": "string",
    "zip": "string"
  },
  "billingAddress": {
    "firstName": "string",
    "lastName": "string",
    "company": "string",
    "address1": "string",
    "address2": "string",
    "city": "string",
    "provinceCode": "string",
    "countryCode": "string",
    "zip": "string"
  },
  "note": "string",
  "sendReceipt": boolean,
  "sendFulfillmentReceipt": boolean
}
```

**Field Details:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `sourceIdentifier` | string | No | - | Unique order ID from your system (for idempotency and tracking) |
| `referringSite` | string (URI) | No | - | Source website or campaign URL |
| `email` | string (email) | No | - | Customer email address |
| `phone` | string | No | - | Customer phone number |
| `financialStatus` | string | No | `"PAID"` | Payment status: `PAID`, `PENDING`, or `AUTHORIZED` (uppercase required) |
| `currency` | string | No | `"USD"` | Three-letter ISO currency code (e.g., `USD`, `CAD`) |
| `lineItems` | array | **Yes** | - | Array of product line items (minimum 1 item) |
| `lineItems[].variantId` | string | **Yes** | - | Shopify variant ID in format: `gid://shopify/ProductVariant/{id}` |
| `lineItems[].quantity` | number | **Yes** | - | Quantity ordered (positive integer, minimum 1) |
| `lineItems[].requiresShipping` | boolean | No | `true` | Whether item requires shipping |
| `shippingAddress` | object | **Yes** | - | Customer shipping address |
| `shippingAddress.firstName` | string | **Yes** | - | First name |
| `shippingAddress.lastName` | string | **Yes** | - | Last name |
| `shippingAddress.company` | string | No | - | Company name |
| `shippingAddress.address1` | string | **Yes** | - | Primary address line |
| `shippingAddress.address2` | string | No | - | Secondary address line (apt, suite, etc.) |
| `shippingAddress.city` | string | **Yes** | - | City name |
| `shippingAddress.provinceCode` | string | **Yes** | - | State/province code (**exactly 2 characters**, e.g., `CA`, `NY`, `ON`) |
| `shippingAddress.countryCode` | string | No | `"US"` | ISO country code (**exactly 2 characters**, e.g., `US`, `CA`) |
| `shippingAddress.zip` | string | **Yes** | - | Postal/ZIP code |
| `billingAddress` | object | **Yes** | - | Billing address (same structure as shipping) |
| `note` | string | No | - | Internal order notes |
| `sendReceipt` | boolean | No | `false` | Send order confirmation email to customer |
| `sendFulfillmentReceipt` | boolean | No | `false` | Send shipping confirmation email to customer |

**Important Notes:**
- Phone numbers are stored at the order level (not in address objects)
- Province and country codes must be **exactly 2 characters**
- Line items do NOT include price (pricing is determined by Shopify product configuration)

**Success Response (201 Created):**

```json
{
  "success": true,
  "order": {
    "id": "uuid",
    "shopifyOrderId": "gid://shopify/Order/1234567890",
    "shopifyOrderName": "#1001",
    "shopifyOrderNumber": "1001",
    "status": "confirmed",
    "createdAt": "2025-12-12T10:00:00.000Z"
  }
}
```

**Error Responses:**

| Status Code | Description | Response Body |
|-------------|-------------|---------------|
| `400 Bad Request` | Validation error | `{ "success": false, "error": "...", "message": "..." }` |
| `401 Unauthorized` | Missing or invalid API key | `{ "success": false, "error": "...", "message": "..." }` |
| `500 Internal Server Error` | Server error | `{ "success": false, "error": "Internal server error", "message": "Failed to create order" }` |
| `502 Bad Gateway` | Shopify API error | `{ "success": false, "error": "Shopify error", "message": "..." }` |

**Example Request:**

```bash
curl -X POST https://api.vitaboom.com/api/vendor/your-vendor/orders \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "sourceIdentifier": "vendor-order-12345",
    "email": "customer@example.com",
    "financialStatus": "PAID",
    "currency": "USD",
    "lineItems": [
      {
        "variantId": "gid://shopify/ProductVariant/48690776727848",
        "quantity": 1,
        "requiresShipping": true
      }
    ],
    "shippingAddress": {
      "firstName": "John",
      "lastName": "Doe",
      "address1": "123 Main St",
      "city": "San Francisco",
      "provinceCode": "CA",
      "countryCode": "US",
      "zip": "94102"
    },
    "billingAddress": {
      "firstName": "John",
      "lastName": "Doe",
      "address1": "123 Main St",
      "city": "San Francisco",
      "provinceCode": "CA",
      "countryCode": "US",
      "zip": "94102"
    }
  }'
```

---

### 3. Retrieve Order

**Endpoint:** `GET /api/vendor/{vendorSlug}/orders/{orderId}`

**Description:** Retrieve complete details for a specific order

**Authentication:** Required

**Path Parameters:**
- `vendorSlug` (string, required) - Your unique vendor identifier
- `orderId` (string, required) - Vitaboom order ID (UUID)

**Success Response (200 OK):**

```json
{
  "success": true,
  "order": {
    "id": "uuid",
    "shopifyOrderId": "gid://shopify/Order/1234567890",
    "shopifyOrderName": "#1001",
    "shopifyOrderNumber": "1001",
    "vendorOrderId": "vendor-order-123",
    "status": "confirmed",
    "financialStatus": "paid",
    "fulfillmentStatus": "unfulfilled",
    "createdAt": "2025-12-12T10:00:00.000Z",
    "updatedAt": "2025-12-12T10:30:00.000Z"
  }
}
```

**Error Responses:**

| Status Code | Description |
|-------------|-------------|
| `401 Unauthorized` | Missing or invalid API key |
| `404 Not Found` | Order not found or doesn't belong to your vendor |
| `500 Internal Server Error` | Server error |

---

### 4. List Orders

**Endpoint:** `GET /api/vendor/{vendorSlug}/orders`

**Description:** Retrieve a paginated list of orders

**Authentication:** Required

**Path Parameters:**
- `vendorSlug` (string, required) - Your unique vendor identifier

**Query Parameters:**

| Parameter | Type | Default | Max | Description |
|-----------|------|---------|-----|-------------|
| `limit` | integer | 50 | 100 | Number of orders to return |
| `offset` | integer | 0 | - | Number of orders to skip (for pagination) |
| `status` | string | - | - | Filter by order status (optional) |

**Status Filter Values:**
- `confirmed`
- `cancelled`
- Other status values as defined by order lifecycle

**Success Response (200 OK):**

```json
{
  "success": true,
  "orders": [
    {
      "id": "uuid",
      "shopifyOrderId": "gid://shopify/Order/1234567890",
      "shopifyOrderName": "#1001",
      "shopifyOrderNumber": "1001",
      "vendorOrderId": "vendor-order-123",
      "status": "confirmed",
      "financialStatus": "paid",
      "fulfillmentStatus": "fulfilled",
      "createdAt": "2025-12-12T10:00:00.000Z",
      "updatedAt": "2025-12-12T12:00:00.000Z"
    }
  ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 150
  }
}
```

**Example Request:**

```bash
curl -X GET "https://api.vitaboom.com/api/vendor/your-vendor/orders?limit=20&offset=0&status=confirmed" \
  -H "X-API-Key: your-api-key"
```

**Error Responses:**

| Status Code | Description |
|-------------|-------------|
| `401 Unauthorized` | Missing or invalid API key |
| `500 Internal Server Error` | Server error |

---

### 5. Get Order Fulfillments

**Endpoint:** `GET /api/vendor/{vendorSlug}/orders/{orderId}/fulfillments`

**Description:** Retrieve fulfillment and tracking information for a specific order

**Authentication:** Required

**Path Parameters:**
- `vendorSlug` (string, required) - Your unique vendor identifier
- `orderId` (string, required) - Vitaboom order ID (UUID)

**Success Response (200 OK):**

```json
{
  "success": true,
  "fulfillments": [
    {
      "id": "gid://shopify/Fulfillment/1234567890",
      "status": "success",
      "trackingCompany": "USPS",
      "trackingNumber": "9400111899562738123456",
      "trackingNumbers": ["9400111899562738123456"],
      "trackingUrl": "https://tools.usps.com/go/TrackConfirmAction?tLabels=9400111899562738123456",
      "trackingUrls": ["https://tools.usps.com/go/TrackConfirmAction?tLabels=9400111899562738123456"],
      "createdAt": "2025-12-12T12:00:00.000Z",
      "updatedAt": "2025-12-12T12:00:00.000Z"
    }
  ]
}
```

**Response - Unfulfilled Order:**

```json
{
  "success": true,
  "fulfillments": []
}
```

**Field Details:**

| Field | Type | Description |
|-------|------|-------------|
| `fulfillments[].id` | string | Shopify fulfillment ID |
| `fulfillments[].status` | string | Fulfillment status (typically `success`) |
| `fulfillments[].trackingCompany` | string \| null | Shipping carrier name (e.g., `USPS`, `FedEx`, `UPS`) |
| `fulfillments[].trackingNumber` | string \| null | Primary tracking number (first in array) |
| `fulfillments[].trackingNumbers` | array | Array of all tracking numbers |
| `fulfillments[].trackingUrl` | string \| null | Primary tracking URL (first in array) |
| `fulfillments[].trackingUrls` | array | Array of all carrier tracking URLs |
| `fulfillments[].createdAt` | string | ISO 8601 timestamp of fulfillment creation |
| `fulfillments[].updatedAt` | string | ISO 8601 timestamp of last update |

**Error Responses:**

| Status Code | Description |
|-------------|-------------|
| `401 Unauthorized` | Missing or invalid API key |
| `404 Not Found` | Order not found |
| `500 Internal Server Error` | Server error |
| `502 Bad Gateway` | Shopify API error |

---

### 6. Update Order

**Endpoint:** `PATCH /api/vendor/{vendorSlug}/orders/{orderId}`

**Description:** Update order details (line items, addresses, notes). Only unfulfilled, non-cancelled orders can be updated. At least one field must be provided.

**Authentication:** Required

**Path Parameters:**
- `vendorSlug` (string, required) - Your unique vendor identifier
- `orderId` (string, required) - Vitaboom order ID (UUID)

**Request Body** (all fields optional, but at least one required):

```json
{
  "lineItems": [
    {
      "variantId": "gid://shopify/ProductVariant/123",
      "quantity": 2,
      "requiresShipping": true
    }
  ],
  "shippingAddress": {
    "firstName": "Jane",
    "lastName": "Smith",
    "company": "Acme Corp",
    "address1": "456 Oak Ave",
    "city": "Los Angeles",
    "provinceCode": "CA",
    "countryCode": "US",
    "zip": "90001"
  },
  "billingAddress": {...},
  "email": "newemail@example.com",
  "phone": "+1-555-0100",
  "note": "Updated delivery instructions"
}
```

**Success Response (200 OK):**

```json
{
  "success": true,
  "order": {
    "id": "uuid",
    "shopifyOrderId": "gid://shopify/Order/1234567890",
    "shopifyOrderName": "#1001",
    "shopifyOrderNumber": "1001",
    "vendorOrderId": "vendor-order-123",
    "status": "confirmed",
    "financialStatus": "paid",
    "fulfillmentStatus": "unfulfilled",
    "createdAt": "2025-12-12T10:00:00.000Z",
    "updatedAt": "2025-12-12T11:00:00.000Z"
  }
}
```

**Error Responses:**

| Status Code | Description |
|-------------|-------------|
| `400 Bad Request` | Validation error (no fields provided or invalid format) |
| `401 Unauthorized` | Missing or invalid API key |
| `404 Not Found` | Order not found |
| `409 Conflict` | Order already fulfilled or cancelled (cannot update) |
| `500 Internal Server Error` | Server error |
| `502 Bad Gateway` | Shopify API error |

---

### 7. Cancel Order

**Endpoint:** `POST /api/vendor/{vendorSlug}/orders/{orderId}/cancel`

**Description:** Cancel an order. Only unfulfilled, non-cancelled orders can be cancelled.

**Authentication:** Required

**Path Parameters:**
- `vendorSlug` (string, required) - Your unique vendor identifier
- `orderId` (string, required) - Vitaboom order ID (UUID)

**Request Body** (optional):

```json
{
  "reason": "CUSTOMER",
  "notifyCustomer": false
}
```

**Field Details:**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `reason` | string | No | - | Cancellation reason: `CUSTOMER`, `FRAUD`, `INVENTORY`, or `OTHER` (uppercase required) |
| `notifyCustomer` | boolean | No | `false` | Send cancellation notification email to customer |

**Success Response (200 OK):**

```json
{
  "success": true,
  "order": {
    "id": "uuid",
    "shopifyOrderId": "gid://shopify/Order/1234567890",
    "shopifyOrderName": "#1001",
    "shopifyOrderNumber": "1001",
    "vendorOrderId": "vendor-order-123",
    "status": "cancelled",
    "financialStatus": "refunded",
    "fulfillmentStatus": "unfulfilled",
    "createdAt": "2025-12-12T10:00:00.000Z",
    "updatedAt": "2025-12-12T11:30:00.000Z"
  }
}
```

**Error Responses:**

| Status Code | Description |
|-------------|-------------|
| `401 Unauthorized` | Missing or invalid API key |
| `404 Not Found` | Order not found |
| `409 Conflict` | Order already cancelled or fulfilled (cannot cancel) |
| `500 Internal Server Error` | Server error |
| `502 Bad Gateway` | Shopify API error |

---

## Webhooks

Vitaboom can send real-time webhook notifications to your system when order events occur. This enables you to maintain synchronized order state without constant polling.

### Setup

Contact Vitaboom support to configure webhooks for your integration:
- **Email:** austin@fractallabs.dev, serhii@fractallabs.dev
- **Required Information:**
  - Your webhook endpoint URL(s) for each event type
  - Optional: Test webhook URLs for non-production environments
  - Optional: Webhook secret for signature verification

### Webhook Event Types

The system supports event-specific webhook URLs. You can configure different endpoints for different event types:

| Event Type | Description | Vendor Configuration Field |
|------------|-------------|---------------------------|
| `orders/create` | Order successfully created and confirmed | `webhookOrderConfirmedUrl` |
| `orders/paid` | Order payment confirmed | `webhookOrderConfirmedUrl` |
| `orders/fulfilled` | Order fulfilled with tracking information | `webhookOrderFulfilledUrl` |
| `orders/cancelled` | Order cancelled | `webhookOrderCancelledUrl` |
| `orders/updated` | Order details modified | `webhookOrderUpdatedUrl` |
| `orders/edited` | Order line items or addresses changed | `webhookOrderUpdatedUrl` |

**Fallback Behavior:** If an event-specific URL is not configured, the system falls back to the general `webhookUrl` field (if configured).

**Test Mode:** In non-production environments, test-specific URLs can be configured via vendor metadata:
- `testWebhookOrderConfirmedUrl`
- `testWebhookOrderFulfilledUrl`
- `testWebhookOrderCancelledUrl`
- `testWebhookOrderUpdatedUrl`

### Webhook Payload Structure

All webhooks follow this consistent structure:

```json
{
  "event": "orders/fulfilled",
  "eventId": "uuid",
  "timestamp": "2025-12-12T12:00:00.000Z",
  "order": {
    "id": "uuid",
    "shopifyOrderId": "gid://shopify/Order/1234567890",
    "shopifyOrderNumber": "1001",
    "shopifyOrderName": "#1001",
    "vendorOrderId": "vendor-order-123",
    "status": "confirmed",
    "financialStatus": "paid",
    "fulfillmentStatus": "fulfilled",
    "createdAt": "2025-12-12T10:00:00.000Z",
    "updatedAt": "2025-12-12T12:00:00.000Z"
  },
  "shopifyData": {
    "id": 1234567890,
    "orderNumber": 1001,
    "name": "#1001",
    "email": "customer@example.com",
    "phone": "+1-555-0100",
    "createdAt": "2025-12-12T10:00:00.000Z",
    "updatedAt": "2025-12-12T12:00:00.000Z",
    "totalPrice": "29.99",
    "currency": "USD",
    "financialStatus": "paid",
    "fulfillmentStatus": "fulfilled",
    "fulfillments": [
      {
        "id": 9876543210,
        "adminGraphqlApiId": "gid://shopify/Fulfillment/9876543210",
        "name": "#1001.1",
        "service": "manual",
        "shipmentStatus": null,
        "status": "success",
        "trackingCompany": "USPS",
        "trackingNumber": "9400111899562738123456",
        "trackingNumbers": ["9400111899562738123456"],
        "trackingUrl": "https://tools.usps.com/go/TrackConfirmAction?tLabels=9400111899562738123456",
        "trackingUrls": ["https://tools.usps.com/go/TrackConfirmAction?tLabels=9400111899562738123456"],
        "createdAt": "2025-12-12T12:00:00.000Z",
        "updatedAt": "2025-12-12T12:00:00.000Z"
      }
    ],
    "lineItems": [
      {
        "id": 1111111111,
        "title": "Product Name",
        "quantity": 1,
        "price": "29.99",
        "sku": "PROD-001",
        "variantId": 48690776727848
      }
    ],
    "shippingAddress": {
      "firstName": "John",
      "lastName": "Doe",
      "address1": "123 Main St",
      "city": "San Francisco",
      "provinceCode": "CA",
      "zip": "94102",
      "countryCode": "US"
    },
    "billingAddress": {...}
  }
}
```

### Webhook Field Details

| Field | Type | Description |
|-------|------|-------------|
| `event` | string | Event type (e.g., `orders/create`, `orders/fulfilled`) |
| `eventId` | string | Unique identifier for this webhook event (UUID - use for deduplication) |
| `timestamp` | string | ISO 8601 timestamp when event occurred |
| `order` | object | Vitaboom order object with current state |
| `shopifyData` | object | Raw Shopify webhook data with additional details |

### Webhook Headers

Each webhook request includes custom headers:

```
Content-Type: application/json
X-Vitaboom-Event: orders/fulfilled
X-Vitaboom-Event-Id: uuid
```

### Event-Specific Payloads

**orders/create & orders/paid:**
- `order.status`: `confirmed`
- `order.fulfillmentStatus`: `unfulfilled`
- `shopifyData.fulfillments`: May be empty array

**orders/fulfilled:**
- `order.fulfillmentStatus`: `fulfilled`
- `shopifyData.fulfillments`: Array with tracking information

**orders/cancelled:**
- `order.status`: `cancelled`
- `order.financialStatus`: `refunded`

**orders/updated & orders/edited:**
- `order`: Contains updated order state
- Modified fields reflected in both `order` and `shopifyData`

### Webhook Retry Logic

Vitaboom automatically retries failed webhook deliveries:

- **Retry Count:** Up to 3 attempts
- **Retry Delay:** Exponential backoff with ~1 second intervals between attempts
- **Timeout:** 10 seconds per request
- **Success Criteria:** HTTP 2xx response status

Failed webhooks are logged and can be monitored via Sentry integration.

### Webhook Best Practices

1. **Idempotency:** Use `eventId` (UUID) to deduplicate webhook deliveries
2. **Response Time:** Respond quickly (< 5 seconds) with `200 OK` to acknowledge receipt
3. **Asynchronous Processing:** Queue webhook processing to avoid timeouts
4. **Retry Tolerance:** Handle duplicate deliveries gracefully (same `eventId` may be received multiple times)
5. **Error Handling:** Log failed processing attempts for debugging
6. **Validation:** Verify webhook authenticity via headers (`X-Vitaboom-Event-Id`)

---

## Rate Limits

To ensure system stability and fair usage, the following rate limits apply:

| Limit Type | Threshold | Reset Period |
|------------|-----------|--------------|
| Standard | 100 requests | Per minute |
| Burst | 10 requests | Per second |

**Rate Limit Exceeded Response:**

```json
{
  "error": "Rate limit exceeded",
  "message": "Too many requests. Please retry after 60 seconds.",
  "retryAfter": 60
}
```

**Status Code:** `429 Too Many Requests`

**Best Practices:**
- Implement exponential backoff for retries
- Use webhooks instead of polling when possible
- Cache responses where appropriate
- Monitor your request volume

---

## Error Handling

### Error Response Format

All errors follow a consistent JSON structure:

```json
{
  "success": false,
  "error": "Error type",
  "message": "Detailed error message"
}
```

### Common Error Codes

| Status Code | Error Type | Description |
|-------------|------------|-------------|
| `400` | Bad Request | Invalid request format or missing required fields |
| `401` | Unauthorized | Missing or invalid API key |
| `404` | Not Found | Resource doesn't exist or not accessible |
| `409` | Conflict | Resource conflict (already cancelled, already fulfilled, etc.) |
| `422` | Unprocessable Entity | Valid format but business logic error |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server-side error (retry with backoff) |
| `502` | Bad Gateway | Upstream service (Shopify) error |
| `503` | Service Unavailable | Temporary service outage |

### Retry Guidelines

| Status Code | Should Retry? | Strategy |
|-------------|---------------|----------|
| `400`, `401`, `404`, `409`, `422` | No | Fix request and resubmit |
| `429` | Yes | Wait for `retryAfter` seconds |
| `500`, `502`, `503` | Yes | Exponential backoff (1s, 2s, 4s, 8s, 16s) |

---

## Data Formats

### Date/Time
All timestamps use ISO 8601 format in UTC:
```
2025-12-12T10:00:00.000Z
```

### Shopify IDs
Shopify resource IDs use Global ID (GID) format:
```
gid://shopify/ProductVariant/48690776727848
gid://shopify/Order/5968471580968
gid://shopify/Fulfillment/1234567890
```

Some numeric IDs may appear in `shopifyData` as plain integers.

### Country Codes
ISO 3166-1 alpha-2 format (**exactly 2 letters**):
```
US, CA, GB, AU
```

### Province/State Codes
ISO 3166-2 subdivision codes (**exactly 2 letters**):
```
CA (California), NY (New York), ON (Ontario), BC (British Columbia)
```

### Currency Codes
ISO 4217 format (3 letters):
```
USD, CAD, EUR, GBP
```

### Currency Values
Currency values appear as strings in API responses:
```json
{
  "totalPrice": "29.99"
}
```

---

## Testing

### Test Mode
Contact support to enable test mode for your integration. Test mode uses separate webhook URLs configured in vendor metadata to prevent production data contamination.

### Sample Test Data

**Test Variant IDs:**
```
gid://shopify/ProductVariant/48690776727848  (Test Product A)
gid://shopify/ProductVariant/48690776727849  (Test Product B)
```

**Test Addresses:**
```json
{
  "firstName": "Test",
  "lastName": "Customer",
  "address1": "123 Test Street",
  "city": "San Francisco",
  "provinceCode": "CA",
  "countryCode": "US",
  "zip": "94102"
}
```

---

## Support & Contact

For technical support, integration assistance, or webhook configuration:

**Email:**
- austin@fractallabs.dev
- serhii@fractallabs.dev

**Response Time:**
- Standard: Within 24 business hours
- Urgent (production issues): Within 4 hours

---

## Changelog

### Version 1.1 (December 2025)
- **Corrected** all response structures to include `{ success: true, ... }` wrapper
- **Corrected** lineItems structure (removed `price` field, added `requiresShipping`)
- **Corrected** phone field location (order-level, not in address objects)
- **Corrected** email field to optional (not required)
- **Corrected** provinceCode and countryCode validation (exactly 2 characters)
- **Corrected** cancel reason values to uppercase (`CUSTOMER`, `FRAUD`, `INVENTORY`, `OTHER`)
- **Corrected** pagination response structure (includes `total`, not `hasMore`)
- **Corrected** fulfillments response to include both singular and plural tracking fields
- **Added** event-specific webhook URL configuration details
- **Added** test mode webhook configuration via metadata
- **Added** webhook retry logic specification
- **Added** requiresShipping field for line items
- **Added** company field for addresses
- Verified against actual implementation in vitaboom-backend codebase

### Version 1.0 (December 2025)
- Initial comprehensive API documentation
- Aggregated endpoints from Humanaut and Thrivelab integrations
- Standardized to generic vendor slug pattern

---

## Appendix: Complete API Reference

### All Available Endpoints

```
GET    /api/health
POST   /api/vendor/{vendorSlug}/orders
GET    /api/vendor/{vendorSlug}/orders
GET    /api/vendor/{vendorSlug}/orders/{orderId}
GET    /api/vendor/{vendorSlug}/orders/{orderId}/fulfillments
PATCH  /api/vendor/{vendorSlug}/orders/{orderId}
POST   /api/vendor/{vendorSlug}/orders/{orderId}/cancel
```

### All Webhook Events

```
orders/create
orders/paid
orders/fulfilled
orders/cancelled
orders/updated
orders/edited
```

---

*This documentation is maintained by Fractal Labs for Vitaboom vendor integrations. Verified against implementation: December 2025*
