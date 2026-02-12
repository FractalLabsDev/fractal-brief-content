# AI Provider Billing API Research

Investigation of programmatic billing, usage, and credit balance access for talkwise-oracle's host providers.

**Providers covered:** OpenAI, Anthropic, xAI (Grok), Groq, Google (Gemini)

---

## Executive Summary

| Provider | Usage API | Cost/Spend API | Credit Balance | Auth Required |
|----------|-----------|----------------|----------------|---------------|
| **OpenAI** | Yes | Yes | Partial | Admin Key |
| **Anthropic** | Yes | Yes | No | Admin Key |
| **xAI (Grok)** | Yes | Yes | Yes | Management Key |
| **Groq** | No | No | No | N/A |
| **Google (Gemini)** | Partial | No | No | Service Account |

**Best case:** xAI is the only provider with full programmatic access to credit balance.

**Worst case:** Groq has no billing API at all.

---

## OpenAI

### Available Endpoints

| What | Endpoint | Auth |
|------|----------|------|
| Token usage by model/project | `GET /v1/organization/usage/completions` | Admin Key |
| Dollar costs | `GET /v1/organization/costs` | Admin Key |
| Credit grants/balance | `GET /v1/dashboard/billing/credit_grants` | Regular Key (unofficial) |

### Usage API
```bash
curl "https://api.openai.com/v1/organization/usage/completions?start_time=1730419200&bucket_width=1d&group_by=model" \
  -H "Authorization: Bearer $OPENAI_ADMIN_KEY"
```

Returns `input_tokens`, `output_tokens`, `input_cached_tokens`, `num_model_requests` grouped by model/project.

### Costs API
```bash
curl "https://api.openai.com/v1/organization/costs?start_time=1730419200&limit=30&group_by=project_id,line_item" \
  -H "Authorization: Bearer $OPENAI_ADMIN_KEY"
```

Returns dollar amounts that reconcile to billing invoices.

### Credit Balance
```bash
curl "https://api.openai.com/v1/dashboard/billing/credit_grants" \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

**Caveat:** This endpoint is semi-unofficial. Community reports mixed reliability.

### Authentication
- **Admin API Key required** for usage/costs endpoints
- Only Organization Owners can create admin keys
- Create at: https://platform.openai.com/settings/organization/admin-keys

### Gotchas
- Credit grants endpoint is not officially documented
- Costs API only supports daily granularity
- Credits expire after 1 year

---

## Anthropic

### Available Endpoints

| What | Endpoint | Auth |
|------|----------|------|
| Token usage | `GET /v1/organizations/usage_report/messages` | Admin Key |
| Dollar costs | `GET /v1/organizations/cost_report` | Admin Key |
| Credit balance | **None** | N/A |

### Usage API
```bash
curl "https://api.anthropic.com/v1/organizations/usage_report/messages?starting_at=2025-01-01T00:00:00Z&ending_at=2025-01-08T00:00:00Z&group_by[]=model&bucket_width=1d" \
  --header "anthropic-version: 2023-06-01" \
  --header "x-api-key: $ANTHROPIC_ADMIN_KEY"
```

Supports grouping by `model`, `workspace_id`, `api_key_id`, `service_tier`.

### Cost API
```bash
curl "https://api.anthropic.com/v1/organizations/cost_report?starting_at=2025-01-01T00:00:00Z&bucket_width=1d" \
  --header "anthropic-version: 2023-06-01" \
  --header "x-api-key: $ANTHROPIC_ADMIN_KEY"
```

Returns costs in USD (cents as decimal strings, e.g., `"123.45"` = $1.23).

### Credit Balance
**No API endpoint exists.** This is a known gap with an open feature request (closed as "Not Planned" for SDK scope).

Workaround: Manual check in [Claude Console Billing](https://console.anthropic.com/settings/billing).

### Authentication
- **Admin API Key required** (`sk-ant-admin...` prefix)
- Standard API keys (`sk-ant-api01...`) will NOT work
- Only organization admins can provision admin keys
- Individual accounts cannot access Admin API

### Gotchas
- No credit balance endpoint at all
- Cost endpoint excludes Priority Tier usage
- Daily granularity only for costs
- Data freshness typically 5 minutes

---

## xAI (Grok)

**Best billing API of the group.** Full programmatic access including credit balance.

### Available Endpoints

| What | Endpoint | Auth |
|------|----------|------|
| Prepaid balance | `GET /v1/billing/teams/{team_id}/prepaid/balance` | Management Key |
| Invoice preview | `GET /v1/billing/teams/{team_id}/postpaid/invoice/preview` | Management Key |
| Invoice history | `GET /v1/billing/teams/{team_id}/invoices` | Management Key |
| Spending limits | `GET /v1/billing/teams/{team_id}/postpaid/spending-limits` | Management Key |
| Top-up credits | `POST /v1/billing/teams/{team_id}/prepaid/top-up` | Management Key |

**Base URL:** `https://management-api.x.ai`

### Credit Balance
```bash
curl -X GET "https://management-api.x.ai/v1/billing/teams/default/prepaid/balance" \
  -H "Authorization: Bearer $XAI_MANAGEMENT_KEY"
```

### Invoice Preview (Current Month Spend)
```bash
curl -X GET "https://management-api.x.ai/v1/billing/teams/default/postpaid/invoice/preview" \
  -H "Authorization: Bearer $XAI_MANAGEMENT_KEY"
```

### Authentication
- **Management Key required** (separate from API key)
- Create at: xAI Console → Settings → Management Keys
- Standard API keys will NOT work for billing endpoints

### Gotchas
- Usage Explorer (detailed per-key/model/IP filtering) is UI-only, not API
- Response schemas not fully documented publicly
- Credit priority: Free → Prepaid → Monthly billing

---

## Groq

**No billing API exists.** This is the worst of the group.

### What's Available
- Per-request `usage` in API responses (tokens, timing)
- Console UI for usage/billing (no API access)

### What's NOT Available
- No usage history endpoint
- No cost/spend endpoint
- No credit balance endpoint
- No spending limits API

### Per-Request Usage (Only Data Point)
API responses include:
```json
{
  "usage": {
    "prompt_tokens": 123,
    "completion_tokens": 456,
    "total_tokens": 579,
    "prompt_time": 0.012,
    "completion_time": 0.034
  }
}
```

### Rate Limit Headers
Every response includes quota headers:
- `x-ratelimit-limit-requests` / `x-ratelimit-remaining-requests`
- `x-ratelimit-limit-tokens` / `x-ratelimit-remaining-tokens`

### Community Status
[Open feature request](https://community.groq.com/t/add-api-endpoint-to-fetch-billing-and-usage-data/378) - Groq acknowledged they're "considering adding an Accounts API" but no timeline.

### Workaround
Must roll your own:
1. Capture `usage` from every API response
2. Multiply by [Groq pricing](https://groq.com/pricing/)
3. Aggregate in your own database

---

## Google (Gemini)

### Direct API Access
**No dedicated billing endpoints** in the Gemini API (`generativelanguage.googleapis.com`).

### Available Alternatives

| Method | What It Provides |
|--------|------------------|
| AI Studio Dashboard | Rate limits, request counts (UI only) |
| Cloud Monitoring API | Historical usage metrics |
| Cloud Billing Export to BigQuery | Detailed cost breakdown (requires setup) |

### Cloud Monitoring Query
```bash
curl -X POST \
  "https://monitoring.googleapis.com/v3/projects/YOUR_PROJECT/timeSeries:query" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -d '{
    "query": "fetch consumer_quota | metric serviceruntime.googleapis.com/quota/allocation/usage | filter resource.service == \"generativelanguage.googleapis.com\" | within 7d"
  }'
```

### Authentication
- Gemini API inference: API key (`x-goog-api-key`)
- Cloud Monitoring: OAuth2 / Service Account with `monitoring.viewer` role
- Cloud Billing: OAuth2 / Service Account with billing permissions

### Gotchas
- No programmatic credit balance check
- Usage data delayed up to 30 minutes
- No rate limit headers in responses (discover limits via 429 errors)
- Quotas are per-project, not per-key

---

## Recommendations for talkwise-oracle Dashboard

### Immediate Actions

1. **Create Admin/Management Keys**
   - OpenAI: Admin Key at [platform.openai.com/settings/organization/admin-keys](https://platform.openai.com/settings/organization/admin-keys)
   - Anthropic: Admin Key at [console.anthropic.com/settings/admin-keys](https://console.anthropic.com/settings/admin-keys)
   - xAI: Management Key at xAI Console → Settings → Management Keys

2. **Build Provider-Specific Integrations**
   | Provider | Approach |
   |----------|----------|
   | OpenAI | Full API integration (usage + costs + partial balance) |
   | Anthropic | Full API integration (usage + costs, no balance) |
   | xAI | Full API integration (all data available) |
   | Groq | Self-track via promptsmith data only |
   | Google | Cloud Monitoring integration or self-track |

3. **Reconciliation Logic**
   - Primary source: promptsmith usage tracking
   - Secondary source: Provider APIs (where available)
   - Alert on discrepancy > threshold (indicates key compromise, bug, or untracked usage)

### Dashboard Data Model

```typescript
interface ProviderStatus {
  provider: 'openai' | 'anthropic' | 'xai' | 'groq' | 'google';

  // From provider APIs (where available)
  actualSpend: number;      // From costs endpoint
  creditBalance: number;    // From balance endpoint (xAI only reliable)

  // From promptsmith
  trackedSpend: number;     // Sum of computed costs

  // Derived
  discrepancy: number;      // actualSpend - trackedSpend
  runway: 'healthy' | 'warning' | 'critical';
}
```

### Fallback Strategy

For providers without balance APIs (Anthropic, Groq, Google):
- Track deposits/top-ups manually or via webhook
- Subtract tracked spend from known balance
- Display as "estimated balance (unverified)"

---

## Source Links

- [OpenAI Usage API](https://platform.openai.com/docs/api-reference/usage)
- [OpenAI Admin API Keys](https://platform.openai.com/docs/api-reference/admin-api-keys)
- [Anthropic Usage/Cost API](https://docs.anthropic.com/en/api/usage-cost-api)
- [Anthropic Admin API](https://docs.anthropic.com/en/api/administration-api)
- [xAI Management API - Billing](https://docs.x.ai/docs/management-api/billing)
- [Groq Billing FAQs](https://console.groq.com/docs/billing-faqs)
- [Google Gemini Rate Limits](https://ai.google.dev/gemini-api/docs/rate-limits)
- [Google Cloud Monitoring API](https://cloud.google.com/monitoring/api/v3)
