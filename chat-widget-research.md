# Research: Chat Widgets for Custom Backend/Webhook Integration

## Summary

The client wants a chat widget for their website that:
1. Connects to a custom n8n workflow via webhook
2. Bot can answer questions with AI
3. Bot can access data and tools
4. Should be open source or free to use

**Answer to the client's question**: Yes, there ARE "empty" widgets that simply forward messages via webhooks to any backend. You don't need to build from scratch - there are ready-made solutions specifically for n8n.

---

## Best Options for n8n Integration

### 1. @n8n/chat (Official n8n Widget) - RECOMMENDED

**Best for**: Quick setup, official support

| Property | Value |
|----------|-------|
| Type | Open Source (MIT) |
| Cost | Free |
| GitHub Stars | Part of n8n repo |
| NPM | `@n8n/chat` |
| CDN | Yes |

**How it works**:
- Embed via CDN or NPM
- Point to your n8n webhook URL
- Messages are sent with `action=sendMessage` and `sessionId`
- Your n8n workflow processes and responds

**Integration code**:
```html
<link href="https://cdn.jsdelivr.net/npm/@n8n/chat/dist/style.css" rel="stylesheet" />
<script type="module">
  import { createChat } from 'https://cdn.jsdelivr.net/npm/@n8n/chat/dist/chat.bundle.es.js';
  createChat({
    webhookUrl: 'https://your-n8n.com/webhook/your-webhook-id',
    mode: 'window', // or 'fullscreen'
    showWelcomeScreen: false,
    initialMessages: ['Hi! How can I help?']
  });
</script>
```

**Pros**:
- Official n8n package
- Streaming support
- Session management
- Customizable styling
- Works with React, Vue, vanilla JS

**Cons**:
- Basic styling (may need CSS tweaks)

---

### 2. n8n-embedded-chat-interface (symbiosika)

**Best for**: Modern web component approach

| Property | Value |
|----------|-------|
| Type | Open Source (Apache 2.0) |
| Cost | Free |
| GitHub | github.com/symbiosika/n8n-embedded-chat-interface |
| Stars | 58 |

**Integration code**:
```html
<script src="https://cdn.jsdelivr.net/npm/n8n-embedded-chat-interface@latest/output/index.js"></script>
<n8n-embedded-chat-interface
  label="AI Assistant"
  hostname="https://your-n8n.com/webhook/id"
  primary-color="#2563eb"
  open-on-start="false">
</n8n-embedded-chat-interface>
```

**Expected request/response**:
```json
// Request to webhook
{ "chatInput": "Hello", "sessionId": "xxx" }

// Response from n8n
{ "output": "Hi! How can I help?", "sessionId": "xxx" }
```

**Pros**:
- Native web component
- Easy theming (colors, dark mode)
- Multilingual (i18n)
- Self-hostable

---

### 3. n8n-chatbot-template (juansebsol)

**Best for**: Full control, self-hosted

| Property | Value |
|----------|-------|
| Type | Open Source (MIT) |
| Cost | Free |
| GitHub | github.com/juansebsol/n8n-chatbot-template |
| Stars | 14 |

**Features**:
- Single JS file, no dependencies
- Session management with UUID
- Custom branding (logo, colors)
- Works on WordPress, Shopify, any HTML

**Pros**:
- Very lightweight
- Full customization
- Self-hosted option

---

## Freemium Services (No-Code)

### 4. N8N Chat UI (n8nchatui.com)

| Property | Value |
|----------|-------|
| Type | Commercial with Free tier |
| Cost | Free to start |
| Features | Visual editor, streaming, file upload, voice chat |

**Pros**:
- No-code visual designer
- Real-time preview
- Streaming responses
- File upload support

---

### 5. ChatWidgetPro (chatwidgetpro.com)

| Property | Value |
|----------|-------|
| Free | 1 widget, limited customization |
| Pro | $19.99/month (8 widgets) |

**Key feature**: Zero data storage - messages go directly to your webhook

---

### 6. Chatrigger (chatrigger.com)

| Property | Value |
|----------|-------|
| Free | 1 chat, 100 messages/month |
| Start | $19/month (3 chats, 2000 msgs) |
| Pro | $59/month (20 chats, 10000 msgs) |

---

## General Open Source Widgets (Any Backend)

### 7. Chatwoot (chatwoot.com)

| Property | Value |
|----------|-------|
| Type | Open Source (MIT) |
| Stars | 26,900 |
| Tech | Ruby on Rails |
| Deployment | Self-hosted or Cloud |

**Best for**: Full customer support platform with AI (Captain)

**Features**:
- Omnichannel (web, email, WhatsApp, FB, Telegram)
- Built-in AI assistant
- Webhook/API for custom integrations
- Self-hosted option

**Cons**:
- Heavy (requires Ruby on Rails knowledge)
- Overkill if you just need a simple widget

---

### 8. ChatKit (github.com/sovaai/chatKit)

| Property | Value |
|----------|-------|
| Type | Open Source (Apache 2.0) |
| Stars | 57 |
| Tech | React + Storeon |

**Best for**: React developers who want a headless widget

Can connect to ANY backend (chatbot, NLP, live chat)

---

### 9. Tiledesk (tiledesk.com)

| Property | Value |
|----------|-------|
| Type | Open Source (MIT) |
| AI Support | Claude, OpenAI, Gemini, LLAMA, Mistral |
| Deployment | On-premise or Cloud |

**Best for**: AI-powered agents with visual no-code builder

---

### 10. Live Helper Chat (livehelperchat.com)

| Property | Value |
|----------|-------|
| Type | Open Source |
| Tech | PHP + Python |
| Stars | ~1.6k |

**Features**:
- Voice, video, screen share
- REST API integrations
- AI chatbots (OpenAI, Rasa, Ollama, Gemini)

---

## Recommendation

### For WheelsFeels (given existing n8n setup):

**Option A - Fastest to implement**:
Use **@n8n/chat** (official widget)
- Add script to website
- Create n8n workflow with Chat Trigger or Webhook
- Connect to existing AI agents

**Option B - More customization**:
Use **n8n-embedded-chat-interface**
- Web component with theming
- Self-host the JS bundle

**Option C - No-code design**:
Use **N8N Chat UI** (n8nchatui.com)
- Visual designer
- Free tier available

### Do NOT build from scratch unless:
- You need very specific features not available in existing solutions
- You want full control over every aspect
- You have developer resources to maintain it

---

## Implementation Steps (if using @n8n/chat)

1. **Create n8n workflow**:
   - Add Webhook Trigger or Chat Trigger node
   - Make it publicly available
   - Add AI Agent node with tools
   - Return response via Respond to Webhook node

2. **Add to website**:
   ```html
   <script type="module">
     import { createChat } from 'https://cdn.jsdelivr.net/npm/@n8n/chat/dist/chat.bundle.es.js';
     createChat({
       webhookUrl: 'https://wheelsfeels.app.n8n.cloud/webhook/chat-widget'
     });
   </script>
   ```

3. **Response format from n8n**:
   ```json
   {
     "output": "Your AI response here",
     "sessionId": "keep-the-session-id"
   }
   ```

---

## Sources

- https://www.npmjs.com/package/@n8n/chat
- https://github.com/symbiosika/n8n-embedded-chat-interface
- https://github.com/juansebsol/n8n-chatbot-template
- https://n8nchatui.com
- https://chatwidgetpro.com
- https://chatrigger.com
- https://chatwoot.com
- https://github.com/sovaai/chatKit
- https://tiledesk.com
- https://livehelperchat.com

---

*Research completed: January 2026*
