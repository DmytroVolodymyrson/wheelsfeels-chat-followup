# WheelsFeels n8n Automation Project

This project contains n8n workflow automations for WheelsFeels, a company selling vehicle storage and sleeping systems.

## Main Workflow: Tawk.to Chat Follow Up Emails

**Workflow ID**: `VU8DVwXsUGUf6qBJ`
**Status**: Active
**Nodes**: 32

### Purpose

Automatically generates and sends personalized follow-up emails to customers who chat on the WheelsFeels website via Tawk.to live chat. The system extracts customer information, matches their vehicle to available products, and sends a professional follow-up email with product recommendations.

---

## Architecture Overview

```
                                    ┌─────────────────────────┐
                                    │   Tawk.to Webhooks      │
                                    └───────────┬─────────────┘
                                                │
              ┌─────────────────────────────────┼─────────────────────────────────┐
              │                                 │                                 │
              ▼                                 ▼                                 ▼
┌─────────────────────────┐   ┌─────────────────────────┐   ┌─────────────────────────┐
│  On New Chat Transcript │   │     On New Ticket       │   │  Form: Upload JSON      │
│  POST /new-chat-transcript│   │  POST /new-ticket       │   │  (Manual Upload)        │
└───────────┬─────────────┘   └───────────┬─────────────┘   └───────────┬─────────────┘
            │                             │                             │
            └─────────────────────────────┼─────────────────────────────┘
                                          │
                                          ▼
                              ┌─────────────────────────┐
                              │  Format Chat Messages   │
                              │  (JavaScript Code)      │
                              └───────────┬─────────────┘
                                          │
                                          ▼
                              ┌─────────────────────────┐
                              │  Information Extractor  │
                              │  (GPT-4.1 LangChain)    │
                              └───────────┬─────────────┘
                                          │
                         ┌────────────────┼────────────────┐
                         │                │                │
                         ▼                ▼                ▼
              ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
              │ Evaluation3  │  │ Skip Email?  │  │Has Contact?  │
              │ (Log to      │  │ (Condition)  │  │(Condition)   │
              │  Sheets)     │  │              │  │              │
              └──────────────┘  └──────┬───────┘  └──────┬───────┘
                                       │                 │
                        ┌──────────────┴──────┐          ▼
                        │                     │   ┌──────────────┐
                        ▼                     ▼   │Create a row3 │
              ┌──────────────────┐    Skip    │   │(Supabase)    │
              │Check Recent Emails│   path    │   └──────────────┘
              │   (Supabase)     │            │
              └────────┬─────────┘            │
                       │                      │
                       ▼                      │
              ┌──────────────────┐            │
              │ Is Not Duplicate?│            │
              │   (If node)      │            │
              └────────┬─────────┘            │
                       │ (not duplicate)      │
                       ▼                      │
              ┌──────────────────┐            │
              │Generate Follow-Up│            │
              │     Email        │            │
              │(GPT-4.1 Agent)   │            │
              │       +          │            │
              │Product Lookup    │            │
              │   AI Tool        │            │
              └────────┬─────────┘            │
                       │                      │
                       ▼                      │
              ┌──────────────────┐            │
              │   Evaluation5    │────────────┘
              │(Check if testing)│
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │Random Wait Time  │
              │   Calculator     │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │   Random Delay   │
              │ (0.1-0.5 hours)  │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │  Gmail: Send     │
              │   a message      │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │Append row sheet1 │
              │ (Google Sheets)  │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │  Create a row2   │
              │   (Supabase)     │
              └──────────────────┘
```

---

## Trigger Sources

### 1. Webhook: On New Chat Transcript
- **Path**: `POST /webhook/new-chat-transcript`
- **Source**: Tawk.to `chat:transcript_created` event
- **Data**: Full chat conversation with messages, visitor info, timestamps

### 2. Webhook: On New Ticket
- **Path**: `POST /webhook/new-ticket`
- **Source**: Tawk.to `ticket:create` event
- **Data**: Ticket subject, message, requester info

### 3. Form Trigger: Upload Tawk.to Chat Transcripts
- **Purpose**: Manual upload of JSON chat files for processing
- **Accepts**: `.json` files

---

## Node Descriptions

### Data Processing Nodes

| Node | Type | Purpose |
|------|------|---------|
| Format Chat Messages | Code (JS) | Formats chat messages into readable transcript with timestamps |
| Edit Fields2/3 | Set | Restructures data for processing |
| Code1 | Code (JS) | Handles multiple file uploads, splits binary data |
| Extract from File1 | Extract From File | Parses uploaded JSON files |

### AI Processing Nodes

| Node | Type | Model | Purpose |
|------|------|-------|---------|
| Information Extractor1 | LangChain Information Extractor | GPT-4.1 (temp: 0.9) | Extracts customer data from chat |
| Generate Follow-Up Email1 | LangChain Agent | GPT-4.1 (temp: 1.5) | Generates personalized email |
| Product Lookup AI Agent Tool1 | LangChain Agent Tool | GPT-4.1 (temp: 0) | Matches vehicle to products via Supabase |
| Email Output Parser1 | Output Parser | - | Structures email output as JSON |
| Find Vehicle Generation | Supabase Tool | - | Maps year/make/model to generation code |
| Search Products by Generation | Supabase Tool | - | Finds products matching generation |

### Conditional Nodes

| Node | Condition | True Path | False Path |
|------|-----------|-----------|------------|
| Skip Email?1 | Email exists AND skip=false | Check duplicates | Check contact info |
| Is Not Duplicate? | No recent email to same address | Generate email | End (silently) |
| Has Contact Info?1 | Email OR phone exists | Log to Supabase | End |
| Evaluation5 | Check if evaluating | Log only | Send email |

### Duplicate Prevention Nodes

| Node | Type | Purpose |
|------|------|---------|
| Check Recent Emails | Supabase (Get Many) | Queries `chat_follow_ups` for emails sent to same address in last hour |
| Is Not Duplicate? | If | Checks if query returned results; blocks duplicate emails |
| Get Current Promo | Supabase | Fetches current promotional offers to include in emails |

### Output Nodes

| Node | Type | Purpose |
|------|------|---------|
| Send a message1 | Gmail | Sends follow-up email to customer |
| Append row in sheet1 | Google Sheets | Logs to "Follow up emails from chat" sheet |
| Create a row2 | Supabase | Logs sent email to `chat_follow_ups` table |
| Create a row3 | Supabase | Logs skipped chats with contact info |

---

## Information Extractor

Extracts the following attributes from chat transcripts:

| Attribute | Type | Description |
|-----------|------|-------------|
| email | string | Customer email address |
| name | string | Customer full name |
| phone_number | number | Phone with country code (default: 1) |
| car_model | string | Vehicle make, model, year, generation |
| product | string | Product/service discussed |
| summary | string | Brief summary of conversation (required) |
| skip | boolean | True if about existing order/refund/return |
| order_number | number | Order number if mentioned |

---

## Email Generation

### System Prompt Rules

1. **Never invent product information** - Only use data from Product Lookup tool
2. **Use Product Link Tool** - Must call tool before recommending products
3. **Product only, no follow-ups** - Don't promise future actions
4. **Proper punctuation** - Questions must end with `?`
5. **No redundancy** - Don't repeat similar phrases
6. **Logical closing** - Match closing to situation
7. **Mandatory signature** - Every email must end with signature

### Email Structure

1. **Subject Line** (40-55 chars)
   - No forbidden words: discount, purchase, offer, off, sale, deal, promo, buy, save
   - Templates with variations for car/name

2. **Greeting** - Variations: "Hi", "Hello", "Hey" with name

3. **Introduction** - Always from "Anton from WheelsFeels"

4. **Product Section** - 4-part description:
   - Part 0: Transition phrase
   - Part 1: Vehicle + product intro
   - Part 2: Benefit sentence 1
   - Part 3: Benefit sentence 2
   - Link on new line

5. **Special Cases**:
   - Vehicle not in database: Offer universal box or Houston scan
   - No vehicle mentioned: Ask what they need
   - Discount questions: "Yes, depends on product"
   - Preowned/budget: Ask about vehicle and product interest

6. **Signature** (mandatory):
   ```
   Best/Thanks/Cheers,
   Anton Lang
   WheelsFeels, Customer Service Specialist
   (866) 665 1911
   ```

---

## Product Lookup System (Supabase-Based)

The Product Lookup AI Agent Tool uses Supabase database queries to match customer vehicles to products dynamically. This replaced the previous hardcoded ~15KB knowledge base in the prompt.

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Product Lookup AI Agent Tool1                         │
│                         (GPT-4.1, temp: 0)                              │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    Uses two Supabase AI Tool nodes:
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         │                                                      │
         ▼                                                      ▼
┌─────────────────────────┐                    ┌─────────────────────────┐
│  Find Vehicle Generation │                    │ Search Products by Gen  │
│  (Supabase Tool)        │                    │  (Supabase Tool)        │
│                         │                    │                         │
│  Table: vehicle_generations                  │  Table: products        │
│  Filters:               │                    │  Filter:                │
│  - brand = Toyota       │  ───────────────▶  │  - car_models contains  │
│  - model_base = RAV4    │  generation_code   │    [brand + generation] │
│  - year_from <= 2022    │                    │                         │
│  - year_to >= 2022      │                    │  Returns: name, url,    │
│                         │                    │  meta_description       │
│  Returns: RAV4 Prime    │                    │                         │
└─────────────────────────┘                    └─────────────────────────┘
```

### Supabase Tables

#### `vehicle_generations` Table

Maps customer vehicle years to generation codes:

| Column | Type | Description |
|--------|------|-------------|
| id | SERIAL | Primary key |
| brand | TEXT | Vehicle make (e.g., "Toyota", "Subaru", "Jeep") |
| model_base | TEXT | Base model name (e.g., "RAV4", "Outback", "Wrangler") |
| generation_code | TEXT | Full generation identifier (e.g., "RAV4 5th Gen", "Wrangler JLU 4-doors") |
| year_from | INTEGER | Start year for this generation |
| year_to | INTEGER | End year for this generation |
| door_count | INTEGER | NULL for any, 2 or 4 for Wranglers |
| variant | TEXT | Special variants (e.g., "Prime", "4xe") |
| notes | TEXT | Additional matching notes |

**Current Vehicle Generations:**

| Brand | Model | Generation | Years | Variant |
|-------|-------|------------|-------|---------|
| Subaru | Outback | 4th Gen | 2010-2014 | - |
| Subaru | Outback | 5th Gen | 2015-2019 | - |
| Subaru | Outback | 6th Gen | 2020-2027 | - |
| Subaru | Forester | 4th Gen | 2014-2018 | - |
| Subaru | Forester | 5th Gen | 2019-2024 | - |
| Subaru | Crosstrek | 2nd Gen | 2018-2023 | - |
| Toyota | 4Runner | 4th Gen | 2003-2009 | - |
| Toyota | 4Runner | 5th Gen | 2010-2024 | - |
| Toyota | 4Runner | 6th Gen | 2025-2027 | - |
| Toyota | RAV4 | 4th Gen | 2013-2018 | - |
| Toyota | RAV4 | 5th Gen | 2019-2027 | - |
| Toyota | RAV4 | RAV4 Prime | 2021-2027 | Prime |
| Jeep | Wrangler | TJ | 1997-2006 | - |
| Jeep | Wrangler | JK 2-doors | 2007-2018 | 2-door |
| Jeep | Wrangler | JKU 4-doors | 2007-2018 | 4-door |
| Jeep | Wrangler | JL 2-doors | 2018-2027 | 2-door |
| Jeep | Wrangler | JLU 4-doors | 2018-2027 | 4-door |
| Jeep | Wrangler | 4xe | 2021-2027 | 4xe |
| Ford | Bronco | 6th Gen | 2021-2027 | - |
| Lexus | GX | 460 | 2010-2023 | - |
| Lexus | GX | 550 | 2024-2027 | - |
| Nissan | Pathfinder | 5th Gen | 2022-2027 | - |
| Nissan | Rogue | 3rd Gen | 2021-2027 | - |
| Toyota | FJ Cruiser | - | 2007-2014 | - |
| Toyota | Land Cruiser | 250 | 2024-2027 | - |
| BMW | 5-series | G30/F90 | 2017-2023 | - |

#### `products` Table

Contains product information with `car_models` JSONB array linking to generations.

### Trim & Variant Stripping

The AI is instructed to strip trim levels and variants before querying the database:

**Terms stripped from model names:**
- **Trims:** Wilderness, TRD Pro, TRD Off-Road, Limited, Premium, SR5, Rubicon, Sahara, Sport, Willys, XLE, SE, SEL, Touring, Onyx, High Altitude, Trail, Nightshade, Adventure, Base, S, X, XT
- **Variants:** Prime, Hybrid, 4xe, PHEV, EV, Electric

**Examples:**
| Customer Input | Searched As | Result |
|----------------|-------------|--------|
| "2023 Outback Wilderness" | model_base="Outback" | Outback 6th Gen |
| "2022 RAV4 Prime" | model_base="RAV4" | RAV4 Prime (variant match) |
| "2024 4Runner TRD Pro" | model_base="4Runner" | 4Runner 5th Gen |
| "2020 Wrangler Rubicon 4-door" | model_base="Wrangler" | Wrangler JLU 4-doors |

### Supabase Tool Node Configurations

#### Find Vehicle Generation

```
Operation: Get Many Rows
Table: vehicle_generations
Filter (PostgREST string):
  brand=eq.{{ $fromAI('brand') }}
  &model_base=eq.{{ $fromAI('model') }}
  &year_from=lte.{{ $fromAI('year') }}
  &year_to=gte.{{ $fromAI('year') }}
Limit: 5
```

#### Search Products by Generation

```
Operation: Get Many Rows
Table: products
Filter (PostgREST string):
  car_models=cs.[{"brand":"{{ $fromAI('brand') }}","model":"{{ $fromAI('generation_code') }}"}]
Returns: name, url, meta_description
```

---

## Product Knowledge Base

Products are stored in the Supabase `products` table and linked to vehicle generations via the `car_models` JSONB array. Organized by vehicle:

### Supported Vehicles (27 types)

| Make | Models |
|------|--------|
| BMW | 5-series (G30/F90) |
| Ford | Bronco 6th Gen 4-door |
| Jeep | Wrangler JK, JKU, JL, JLU, 4xe, TJ |
| Lexus | GX460, GX550 |
| Nissan | Pathfinder 5th Gen, Rogue 3rd Gen |
| Subaru | Crosstrek 2nd Gen, Forester 4th/5th/6th Gen, Outback 4th/5th/6th Gen |
| Toyota | 4Runner 4th/5th/6th Gen, RAV4 4th/5th Gen, FJ Cruiser, Land Cruiser 250 |
| Universal | Portable Car Kitchen "Voyageurs" |

### Product Types (50+)
- Storage Drawer Systems
- Camping Drawer Systems
- Storage & Camping Systems
- Pull-Out Platforms
- Trunk Mounting Bases
- Spare Tire Well Organizers
- Tailgate Tables
- Shelves
- DIY Precut Kits

---

## Scheduling Logic

The Random Wait Time Calculator implements smart delay:

```javascript
// Timezone: America/Chicago (CST)
// Daytime delay: 0.1 - 0.5 hours (6-30 minutes)
// Night hours: 23:00 - 04:59 (emails not sent)
// If calculated time falls in night: reschedule to 06:00 + 1-2 hours
```

This makes emails appear naturally timed rather than instant automated responses.

---

## Duplicate Email Prevention

Prevents sending multiple follow-up emails when a customer opens multiple chat sessions in quick succession.

### Configuration

| Setting | Value |
|---------|-------|
| Time Window | 1 hour |
| Match Criteria | Email address only |
| Action on Duplicate | Silently skip (no logging) |

### How It Works

1. After `Skip Email?1` determines an email should be sent, the workflow queries Supabase
2. **Check Recent Emails** node searches `chat_follow_ups` table for:
   - Same `customer_email` as current chat
   - `sent_at` within the last hour
3. **Is Not Duplicate?** node evaluates the query results:
   - If no results (first email): Continue to generate and send email
   - If results found (duplicate): End workflow silently

### Filter Query

```
customer_email=eq.{{ $json.output.email }}&sent_at=gt.{{ $now.minus(1, 'hour').toISO() }}
```

This leverages the existing `chat_follow_ups` table that already logs all sent emails, requiring no additional database tables.

---

## Data Storage

### Google Sheets
- **Document**: "Test chatbot new" (`1JClbNIx5HzN1Kt4L3voHF229eSYUETTO2zkGdWoUOLc`)
- **Sheet**: "Follow up emails from chat" (gid: 1447369663)
- **Columns**: chat_transcript, name, email, phone_number, car_model, product_mentioned, conversation_summary, generated_subject, generated_email, date

### Supabase

**Table: `chat_follow_ups`**
- **Fields**: customer_name, customer_email, customer_phone, car_model, product, chat_summary, email_subject, email_body, sent_at, order_number

**Table: `vehicle_generations`**
- **Fields**: id, brand, model_base, generation_code, year_from, year_to, door_count, variant, notes, created_at
- **Purpose**: Maps customer vehicle years to product generation codes

**Table: `products`**
- **Fields**: id, name, url, meta_description, car_models (JSONB array), ...
- **Purpose**: Product catalog with vehicle compatibility via car_models array

---

## Credentials

| Service | Credential Name | Used For |
|---------|-----------------|----------|
| Google Sheets | Google Sheets account | Read/write spreadsheet data |
| Gmail | Gmail account | Send follow-up emails |
| OpenAI | OpenAi WheelsFeels | GPT-4.1 for all AI nodes |
| Supabase | WheelsFeels Supabase | Database logging |

---

## Webhook URLs

Production webhooks (on n8n cloud):
- `https://wheelsfeels.app.n8n.cloud/webhook/new-chat-transcript`
- `https://wheelsfeels.app.n8n.cloud/webhook/new-ticket`

---

## Related Files

- [knowledge-base-plan.md](knowledge-base-plan.md) - Original plan for migrating product knowledge base to Supabase database

---

## Completed Improvements

1. ✅ Migrated product lookup from hardcoded prompt (~15KB) to Supabase database
2. ✅ Created `vehicle_generations` table with year-to-generation mappings
3. ✅ Implemented two Supabase AI Tool nodes for dynamic product lookup
4. ✅ Added trim/variant stripping in AI prompt (handles Wilderness, TRD Pro, Prime, etc.)

---

## Future Improvements

1. Auto-sync `vehicle_generations` from a master spreadsheet
2. Add product type filtering (storage vs camping)
3. Create admin interface for managing vehicle generations
4. Add fuzzy matching for model names ("Rav4" vs "RAV4")
5. Create `vehicle_trims` lookup table for more robust trim handling (if AI prompt approach proves insufficient)
6. Optional: Auto-sync products with website product data
