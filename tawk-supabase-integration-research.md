# Research: Tawk.to AI Assistant + Supabase Integration

## Executive Summary

**YES, it is possible to integrate Tawk.to AI Assistant with Supabase for real-time database queries.**

Tawk.to AI Assist natively supports custom API integrations through OpenAPI specifications. Supabase provides multiple ways to expose data as APIs that can be consumed by Tawk.to.

---

## Key Research Findings

### Tawk.to AI Assist - Custom API Integration Capabilities

Tawk.to's AI Assist supports **custom API tools** that allow the AI chatbot to query external systems in real-time.

**Official Documentation:** https://help.tawk.to/article/how-to-set-up-a-custom-api-integration-with-apollo-ai

**Requirements:**
1. **OpenAPI 3.0 Schema** - JSON or YAML file defining your API endpoints, parameters, and responses
2. **Publicly Hosted Schema** - Must be accessible via URL (GitHub, S3, any web server)
3. **API Base URL** - The root URL where your API endpoints are available
4. **Authentication** - Supports: No Auth, API Key, or Basic Auth

**How It Works:**
- You provide an OpenAPI spec that describes what data is available
- The AI learns from the schema descriptions when to call which endpoint
- When a customer asks a relevant question, the AI automatically calls your API
- The AI uses the response to generate an answer

**Configuration Location:** Tawk.to Dashboard → Add-ons → AI Assist → Settings → Integration/API → Add Tool

---

### Supabase API Options

Supabase provides **three ways** to expose data as APIs:

#### Option 1: Auto-Generated REST API (PostgREST)
- **URL Format:** `https://<project_ref>.supabase.co/rest/v1/<table_name>`
- **OpenAPI Spec Available:** `https://<project_ref>.supabase.co/rest/v1/` (with trailing slash)
- **Authentication:** Requires `apikey` header
- **Pros:** Zero code, instant API for all tables
- **Cons:** Exposes raw table structure, limited control over response format

#### Option 2: Supabase Edge Functions (Recommended)
- **URL Format:** `https://<project_ref>.supabase.co/functions/v1/<function-name>`
- **Runtime:** Deno/TypeScript, globally distributed
- **Pros:**
  - Full control over API structure and response format
  - Can run custom business logic
  - Can aggregate data from multiple tables
  - Can format responses specifically for AI consumption
- **Cons:** Requires writing TypeScript code

#### Option 3: Database Functions (RPC)
- **URL Format:** `https://<project_ref>.supabase.co/rest/v1/rpc/<function-name>`
- **Pros:** SQL logic stays in database, fast execution
- **Cons:** Complex logic harder to maintain in SQL

---

## Recommended Integration Architecture

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│   Tawk.to AI    │────▶│  Supabase Edge       │────▶│    Supabase     │
│   Assistant     │     │  Function(s)         │     │    Database     │
│                 │◀────│                      │◀────│    (Postgres)   │
└─────────────────┘     └──────────────────────┘     └─────────────────┘
        │                         │
        │                         │
        ▼                         ▼
   OpenAPI Spec              Custom Logic
   (hosted on GitHub)        (product lookup,
                             inventory check,
                             price formatting)
```

**Why Edge Functions are Best:**
1. Can format responses optimally for AI consumption
2. Can include business logic (discounts, availability, shipping info)
3. Can combine data from multiple tables in one response
4. Can add caching for frequently requested data
5. Can sanitize/limit data exposure (security)

---

## Implementation Steps

### Step 1: Create Supabase Edge Function(s)

Example function for product lookup:
```typescript
// supabase/functions/product-lookup/index.ts
import { createClient } from 'npm:@supabase/supabase-js@2'

Deno.serve(async (req) => {
  const { vehicle, product_type } = await req.json()

  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )

  const { data, error } = await supabase
    .from('products')
    .select('name, description, price, url, in_stock')
    .ilike('vehicle', `%${vehicle}%`)
    .ilike('category', `%${product_type}%`)

  if (error) {
    return new Response(JSON.stringify({ error: error.message }), { status: 500 })
  }

  return new Response(JSON.stringify({ products: data }), {
    headers: { 'Content-Type': 'application/json' }
  })
})
```

### Step 2: Create OpenAPI 3.0 Specification

```yaml
openapi: 3.0.0
info:
  title: WheelsFeels Product API
  description: API for querying WheelsFeels product catalog
  version: 1.0.0
servers:
  - url: https://<project_ref>.supabase.co/functions/v1
paths:
  /product-lookup:
    post:
      summary: Look up products by vehicle and type
      description: Returns matching products from the WheelsFeels catalog
      operationId: lookupProducts
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                vehicle:
                  type: string
                  description: "Vehicle make, model, year (e.g., 'Toyota 4Runner 5th Gen')"
                product_type:
                  type: string
                  description: "Type of product (e.g., 'storage drawer', 'camping system')"
      responses:
        '200':
          description: List of matching products
          content:
            application/json:
              schema:
                type: object
                properties:
                  products:
                    type: array
                    items:
                      type: object
                      properties:
                        name:
                          type: string
                        description:
                          type: string
                        price:
                          type: number
                        url:
                          type: string
                        in_stock:
                          type: boolean
```

### Step 3: Host OpenAPI Spec
- Upload to GitHub Gist (click "Raw" to get URL)
- Or host in Supabase Storage
- Or any public URL

### Step 4: Configure Tawk.to AI Assist
1. Go to Tawk.to Dashboard → Add-ons → AI Assist
2. Select your AI Agent → Integration/API tab
3. Click "Add Tool"
4. Enter:
   - **Schema File URL:** Your OpenAPI spec URL
   - **API Base URL:** `https://<project_ref>.supabase.co/functions/v1`
   - **Authentication:** API Key (use Supabase anon key or service key)
5. Save

### Step 5: Update Base Prompt
Add instructions to the AI's base prompt:
```
When a customer asks about products for their vehicle:
1. Use the product-lookup API to find matching products
2. Always include the product URL in your response
3. Mention if a product is out of stock
```

---

## Alternative: No-Code Integration via n8n

If you prefer a no-code approach, you could:
1. Create an n8n workflow that queries Supabase
2. Expose it via n8n webhook
3. Create OpenAPI spec for the n8n webhook
4. Connect to Tawk.to

---

## Limitations & Considerations

1. **API Call Limits:** Each Tawk.to AI message that triggers an API call uses message credits
2. **Latency:** API calls add latency to responses (~200-500ms)
3. **API Key Security:** Tawk.to stores your API key securely, but the API itself should have rate limiting
4. **Response Size:** Keep API responses concise for faster AI processing
5. **Paid Feature:** API integrations require a paid Tawk.to AI Assist plan

---

## Sources

- [Tawk.to Custom API Integration Guide](https://help.tawk.to/article/how-to-set-up-a-custom-api-integration-with-apollo-ai)
- [Tawk.to AI Assist Getting Started](https://help.tawk.to/article/getting-started-with-ai-assist)
- [Supabase Edge Functions Docs](https://supabase.com/docs/guides/functions)
- [Supabase REST API Docs](https://supabase.com/docs/guides/api)
- [Tawk.to API Integration Samples (GitHub)](https://github.com/tawk/api-integration-samples)
- [Tawk.to AI Assist YouTube Tutorial](https://www.youtube.com/watch?v=i1LJy87pQTo)
- [Tawk.to API Integrations Video](https://www.youtube.com/watch?v=EvhPeJErJqQ)
