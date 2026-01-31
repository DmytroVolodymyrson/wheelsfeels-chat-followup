# WheelsFeels Chat Follow-up

Automated email follow-ups for WheelsFeels live chat conversations.

## What it does

When a customer chats on the WheelsFeels website via Tawk.to, this n8n workflow:

1. Receives the chat transcript via webhook
2. Extracts customer info and vehicle details using AI
3. Matches their vehicle to available products in the database
4. Generates and sends a personalized follow-up email

## Tech Stack

- **n8n** — Workflow automation
- **Supabase** — Product database & vehicle generation mappings
- **OpenAI GPT-4** — Information extraction & email generation
- **Tawk.to** — Live chat widget (webhook source)
- **Gmail** — Email delivery

## Project Structure

```
├── CLAUDE.md                 # Detailed technical documentation
├── prompts/                  # AI prompt templates
├── test_chat_transcripts.json # Test cases
└── *.md                      # Research & planning docs
```

## Setup

1. Configure n8n credentials (Gmail, Supabase, OpenAI)
2. Set up Tawk.to webhook to point to n8n
3. Populate `vehicle_generations` table in Supabase

See `CLAUDE.md` for detailed configuration.
