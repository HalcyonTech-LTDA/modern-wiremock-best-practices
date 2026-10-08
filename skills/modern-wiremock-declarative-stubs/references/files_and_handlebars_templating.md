# External Payloads (`__files/`) & Handlebars Response Templating Guide

WireMock separates request routing from response content through the `__files/` directory, and enables dynamic mock generation via its built-in Handlebars templating engine (`response-template`).

---

## 1. The `__files/` Directory Architecture

The `__files/` directory stores static response payloads, templated documents, and binary files.

### Directory Organization:
Always organize `__files/` symmetrically with `mappings/`, using modular subdirectories for distinct domains or microservices:

```text
wiremock/
├── mappings/
│   ├── auth/
│   │   ├── token-exchange.json
│   │   └── userinfo.json
│   └── billing/
│       ├── invoice-pdf.json
│       └── subscription-lookup.json
└── __files/
    ├── auth/
    │   ├── token-exchange-response.json
    │   └── userinfo-response.json
    └── billing/
        ├── sample-invoice.pdf
        └── subscription-active.json
```

### Linking Mapping to Payload:
In the mapping definition, specify `bodyFileName` with a path **relative to `__files/`**:

```json
{
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json; charset=UTF-8"
    },
    "bodyFileName": "billing/subscription-active.json"
  }
}
```

*Note: Never include leading slashes (`/billing/...` ❌) or the `__files/` prefix (`__files/billing/...` ❌) in `bodyFileName`.*

---

## 2. Enabling Handlebars Response Templating

Dynamic templating transforms static fixtures into intelligent, contextual responses without writing custom Java extensions.

To enable Handlebars processing for a stub, declare the `response-template` transformer inside the `response` block:

```json
{
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json; charset=UTF-8"
    },
    "bodyFileName": "orders/order-details.json",
    "transformers": ["response-template"]
  }
}
```

---

## 3. Request Model Inspection

The `request` object exposes all attributes of the incoming HTTP transaction:

### 1. Path Segments: `request.pathSegments.[index]`
Path segments are 0-indexed strings extracted from the URL path.
For URL `/api/v1/tenants/acme-corp/users/usr-42`:
- `{{request.pathSegments.[0]}}` -> `api`
- `{{request.pathSegments.[1]}}` -> `v1`
- `{{request.pathSegments.[2]}}` -> `tenants`
- `{{request.pathSegments.[3]}}` -> `acme-corp`
- `{{request.pathSegments.[4]}}` -> `users`
- `{{request.pathSegments.[5]}}` -> `usr-42`

### 2. Query Parameters: `request.query.<name>`
Access individual query parameters:
```handlebars
"requestedPage": {{request.query.page}},
"searchFilter": "{{request.query.filter}}"
```

### 3. Request Headers: `request.headers.<name>`
Extract headers (case-insensitive header lookup):
```handlebars
"clientCorrelationId": "{{request.headers.X-Correlation-ID}}",
"userAgent": "{{request.headers.User-Agent}}"
```

### 4. JSON Request Body: `jsonPath`
Extract deeply nested attributes from incoming JSON payloads:
```handlebars
"customerId": "{{jsonPath request.body '$.customer.id'}}",
"accountType": "{{jsonPath request.body '$.customer.account.type'}}"
```

### ⚠️ Critical Rule: Double Braces vs Triple Braces
Handlebars by default HTML-escapes string content (turning `"` into `&quot;`, `<` into `&lt;`).
When rendering JSON objects, complex arrays, or unescaped strings, **ALWAYS use triple braces `{{{ ... }}}`**:

```json
{
  "rawMetadata": {{{jsonPath request.body '$.metadata'}}},
  "sanitizedString": "{{{jsonPath request.body '$.description'}}}"
}
```

---

## 4. Dynamic Data Generation Helpers

WireMock bundles rich Handlebars helper functions for synthesizing dynamic test data:

### Random Value Generator: `randomValue`
```handlebars
"id": "{{randomValue type='UUID'}}",
"sku": "{{randomValue type='ALPHANUMERIC' length=8 uppercase=true}}",
"pinCode": "{{randomValue type='NUMERIC' length=6}}",
"checksum": "{{randomValue type='HEXADECIMAL' length=16}}"
```

### Date & Time Helpers: `now`
Generate ISO-8601 timestamps, epoch milliseconds, or relative offset dates:
```handlebars
"timestamp": "{{now}}",
"issueDate": "{{now format='yyyy-MM-dd'}}",
"isoTimestamp": "{{now format=\"yyyy-MM-dd'T'HH:mm:ss.SSSXXX\"}}",
"expiryDate": "{{now offset='30 days' format='yyyy-MM-dd'}}",
"validUntil": "{{now offset='2 hours' format=\"yyyy-MM-dd'T'HH:mm:ssXXX\"}}"
```

### Math & String Manipulation Helpers:
```handlebars
"itemCount": {{math request.query.count '+' 1}},
"uppercaseName": "{{capitalize (request.query.name)}}",
"slug": "{{lower (request.pathSegments.[2])}}"
```

---

## 5. Conditionals & Iteration

Model dynamic response variations in a single fixture:

### Conditional Logic (`#if`, `#eq`):
```json
{
  "tier": "{{request.query.tier}}",
  "discountPercent": {{#if (eq request.query.tier 'VIP')}}25{{else}}0{{/if}},
  "accessGranted": {{#if (eq request.headers.X-Api-Key 'secret-key')}}true{{else}}false{{/if}}
}
```

### Array Iteration (`#each`):
```json
{
  "orderId": "{{randomValue type='UUID'}}",
  "processedItems": [
    {{#each (jsonPath request.body '$.items')}}
    {
      "sku": "{{this.sku}}",
      "status": "ALLOCATED"
    }{{#unless @last}},{{/unless}}
    {{/each}}
  ]
}
```

---

## 6. Multi-Format Payloads & Binary Assets

A major architectural advantage of delegating response payloads to `__files/` is native support for any content type beyond JSON:

### 1. Binary Files (PDFs, Images, ZIP):
WireMock streams raw bytes without corruption:
- Mapping declares appropriate MIME type: `Content-Type: application/pdf`.
- File placed in `__files/invoices/invoice-sample.pdf`.
- `bodyFileName: "invoices/invoice-sample.pdf"`.

### 2. SOAP & XML Payloads:
- Mapping declares: `Content-Type: application/xml; charset=UTF-8`.
- File placed in `__files/soap/customer-response.xml`.
- Handlebars expressions can extract XML data using `{{{xPath request.body '/soap:Envelope/soap:Body/id/text()' }}}`.

### 3. CSV & Plain Text:
- Mapping declares: `Content-Type: text/csv`.
- File placed in `__files/reports/export.csv`.

---

## 7. Declarative Webhooks in WireMock 3.x

Modern WireMock 3.x natively supports outbound webhooks triggered automatically upon matching a request:

```json
{
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/payments/process"
  },
  "response": {
    "status": 202,
    "headers": { "Content-Type": "application/json; charset=UTF-8" },
    "bodyFileName": "payments/payment-accepted.json"
  },
  "postServeActions": [
    {
      "name": "webhook",
      "parameters": {
        "method": "POST",
        "url": "{{jsonPath request.body '$.callbackUrl'}}",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": "{\"paymentId\": \"{{randomValue type='UUID'}}\", \"status\": \"SETTLED\"}"
      }
    }
  ]
}
```

---

## 8. Anti-Patterns to Avoid

| Anti-Pattern | Why It Fails | Modern Fix |
| :--- | :--- | :--- |
| **Inlined Payloads** (`"body": "{ ... }"`) | Pollutes `mappings/` files, invalidates JSON linting, breaks binary payloads, obscures Git diffs. | Move to `__files/` and link via `bodyFileName`. |
| **HTML Entity Corruption** (`{{jsonPath ...}}`) | Turns quotes into `&quot;`, invalidating client JSON parsing. | Use triple braces `{{{jsonPath ...}}}` for unescaped content. |
| **Missing Transformer** | Outputs raw string literal `{{request.pathSegments.[0]}}` to client. | Add `"transformers": ["response-template"]` to mapping. |
| **Hardcoded Static Clones** | Maintaining 50 stub files for 50 IDs. | Use a single regex `urlPathPattern` with dynamic `request.pathSegments.[n]`. |
