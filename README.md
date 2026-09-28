# MemoryDesk

MemoryDesk is an AI-powered customer support application designed around persistent customer memory.

The key architectural distinction is:

> Session history answers "What are we discussing right now?"
> Long-term memory answers "What should we remember about this customer?"

## Features

- Customer selector
- AI-generated support responses
- Groq using an OpenAI-compatible client
- Hindsight long-term memory
- Separate memory bank per customer
- Current session history
- Memory ON/OFF toggle
- Retrieved-memory panel
- Customer memory count
- Session reset
- Health/status indicator
- Hindsight failure fallback
- Asynchronous memory retention
- Clear memory trace

## Architecture

```text
Customer Message
      |
      v
Current Session
      |
      v
Hindsight Recall
      |
      v
Relevant Memories
      |
      +----------------+
      |                |
      v                |
Session Context       |
      |                |
      +-------+--------+
              |
              v
           Groq LLM
              |
              v
        Support Response
              |
              v
       Hindsight Retain
          (async)
```

## Session vs Long-Term Memory

Session history is held in `sessions/sessionStore.js` and is intentionally temporary.

Hindsight is isolated in `memory/hindsight.js` and is intended for durable customer information such as:

- Previous incidents
- Environment details
- Previous resolutions
- Recurring problems
- Customer context

Resetting a session does not delete Hindsight memory.

## Installation

Requires Node.js 18+.

```bash
npm install
```

Create `.env`:

```bash
cp .env.example .env
```

Set:

```env
GROQ_API_KEY=your_key
GROQ_MODEL=llama-3.3-70b-versatile

HINDSIGHT_BASE_URL=http://localhost:8888
HINDSIGHT_API_KEY=
```

## Run

```bash
npm start
```

Open:

```text
http://localhost:3000
```

Development mode:

```bash
npm run dev
```

## API

### Customers

```http
GET /api/customers
```

### Chat

```http
POST /api/chat
Content-Type: application/json
```

Example:

```json
{
  "customerId": "customer_001",
  "message": "The API is timing out again.",
  "memoryEnabled": true
}
```

### Reset session

```http
POST /api/chat/reset
Content-Type: application/json
```

```json
{
  "customerId": "customer_001"
}
```

### Health

```http
GET /api/health
```

## Hindsight integration

All Hindsight-specific HTTP behavior is isolated in:

```text
memory/hindsight.js
```

This keeps the rest of the application independent from the memory provider.

If the Hindsight service is unavailable:

1. Recall returns an empty memory list.
2. The LLM still receives current session context.
3. The response is generated normally.
4. The UI reports that memory is unavailable.
5. The application remains usable.

If your Hindsight deployment exposes different REST endpoint paths or payloads, update only `memory/hindsight.js`.

## Groq

Groq is accessed through the OpenAI-compatible JavaScript client:

```js
new OpenAI({
  apiKey: process.env.GROQ_API_KEY,
  baseURL: "https://api.groq.com/openai/v1"
});
```

The rest of the application does not need to know the LLM provider.

## Project structure

```text
MemoryDesk/
├── routes/
│   ├── chat.js
│   ├── customers.js
│   └── health.js
├── memory/
│   └── hindsight.js
├── llm/
│   └── groq.js
├── models/
│   ├── customer.js
│   └── conversation.js
├── sessions/
│   └── sessionStore.js
├── public/
│   ├── index.html
│   ├── style.css
│   └── app.js
├── data/
│   └── customers.json
├── server.js
├── package.json
└── README.md
```

## GitHub

```bash
git init
git add .
git commit -m "Initial MemoryDesk implementation"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/MemoryDesk.git
git push -u origin main
```

Never commit `.env`.

## Production considerations

This prototype intentionally keeps session state in process memory.

For production:

- Use Redis or another shared session store.
- Add authentication and authorization.
- Encrypt sensitive customer information.
- Add structured logging.
- Add request validation/rate limiting.
- Add retries and circuit breaking around Hindsight.
- Add observability for recall latency and retain failures.
- Use a persistent customer/profile database.
- Add automated tests.
- Pin and regularly update dependencies.
