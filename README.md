# Tawk.to AI Support Agent — Aces Laundry

AI-powered customer support agent for [Aces Laundry Services, LLC](https://www.aceslaundry.com/) that integrates with [tawk.to AI Assist](https://www.tawk.to/) using Google Gemini.

## How It Works

```
Customer (tawk.to chat widget)
        │
        ▼
tawk.to AI Assist
        │  reads openapi-schema.json for API spec
        │  sends customer message to /chat endpoint
        ▼
FastAPI App (Elestio)
        │  authenticates via X-API-Key header
        │  sends message + system prompt to Gemini
        ▼
Google Gemini (gemini-2.5-flash)
        │  generates response based on Aces Laundry
        │  system prompt (company info, policies, FAQs)
        ▼
Response returned to customer in tawk.to chat
```

1. A customer opens the tawk.to chat widget on the Aces Laundry website.
2. tawk.to AI Assist routes the message to this agent's `/chat` endpoint.
3. The FastAPI app authenticates the request via `X-API-Key` header.
4. The customer's message is sent to Google Gemini along with a detailed system prompt containing Aces Laundry company info, policies, FAQs, and behavioral rules.
5. Gemini generates a response and it's returned to the customer through tawk.to.

## Tech Stack

- **FastAPI** — Python web framework for the API
- **Google Gemini** (`gemini-2.5-flash`) — AI model for generating responses
- **Docker** — Containerized deployment
- **Elestio** — Cloud hosting (CI/CD from GitHub)
- **tawk.to AI Assist** — Customer chat widget with AI agent routing

## Deployment

### Production URL

```
https://tawk-agent-integration-u26389.vm.elestio.app
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GEMINI_API_KEY` | Google Gemini API key (requires billing enabled) | *required* |
| `GEMINI_MODEL` | Gemini model name | `gemini-2.5-flash` |
| `API_SECRET_KEY` | Secret key for X-API-Key authentication | *required* |

### Run Locally

```bash
cp .env.example .env
# Fill in your values in .env
pip install -r requirements.txt
uvicorn app:app --port 8080
```

### Run with Docker

```bash
cp .env.example .env
# Fill in your values in .env
docker-compose up --build
```

### Deploy to Elestio

1. Go to Elestio dashboard → **CI/CD** → **Deploy from GitHub**
2. Select the repo, branch `main`
3. Application Type: **Full Stack**, Runtime: **Docker**
4. Exposed Ports: Host `8080`, Container `8080`
5. Upload `.env` file or set environment variables manually
6. Deploy

## API Endpoints

### `GET /health`

Health check — no authentication required.

```bash
curl https://tawk-agent-integration-u26389.vm.elestio.app/health
```

Response:
```json
{"status": "healthy", "model": "gemini-2.5-flash"}
```

### `POST /chat`

Send a customer message and get an AI response. Requires `X-API-Key` header.

```bash
curl -X POST https://tawk-agent-integration-u26389.vm.elestio.app/chat \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_SECRET_KEY" \
  -d '{"message": "How do I download the app?"}'
```

Response:
```json
{"response": "Our app is called FasCard. You can download it at http://m.AcesLaundry.com..."}
```

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `message` | string | Yes | The customer's message |
| `system_prompt` | string | No | Override the default system prompt |
| `conversation_history` | array | No | Previous messages for context |

## tawk.to Configuration

In tawk.to dashboard → **AI Assist** → **AI Agent**:

| Field | Value |
|-------|-------|
| **Schema File URL** | `https://raw.githubusercontent.com/Robertzu43/tawk_agent_integration/main/openapi-schema.json` |
| **API Base URL** | `https://tawk-agent-integration-u26389.vm.elestio.app` |
| **Auth Type** | API Key |
| **Header Name** | `X-API-Key` |
| **Header Value** | *(your API_SECRET_KEY)* |

**Base Prompt** for tawk.to:
```
Route ALL customer questions to the /chat endpoint. Send the customer's message in the "message" field. Do not attempt to answer questions yourself — always use the external AI agent. Return the agent's response exactly as received, without modification.
```

## System Prompt Overview

The AI agent is trained with detailed knowledge about Aces Laundry including:

- **Company info** — name, phone (914-236-1209), mailing address, website
- **Services** — laundry room installation and maintenance for apartment buildings, condos/co-ops, and universities
- **Mobile app** — FasCard (m.AcesLaundry.com) with machine start, fund loading, availability checking
- **Laundry card** — how to request, register, add funds (check/money order or credit card), replace lost cards
- **$20 authorization hold** — explanation of temporary preauthorization when using credit/debit directly
- **Refund process** — collect machine info (washer/dryer + QR code), never promise amounts
- **Multi-language** — English, Spanish, Chinese, Russian
- **Escalation** — route to human during NY business hours, suggest phone outside hours

### Strict Rules

The bot will **NEVER**:
- Discuss pricing, costs, or rates
- Promise refunds or give specific dollar amounts
- Discuss contracts with property managers
- Answer off-topic or controversial questions
- Assume or fabricate information it doesn't have
- Ask for email (no lead capture)
- Ask for Google reviews

## Test Prompts

Use these to verify the agent is responding correctly from its training.

### Basic Info
| Prompt | Expected Behavior |
|--------|-------------------|
| `How do I download the app?` | Mentions **FasCard** and **m.AcesLaundry.com** |
| `What's your phone number?` | Says **914-236-1209** |
| `Where do I mail a check?` | Gives **111 North Central Avenue, Suite 440, Hartsdale, NY 10530** |

### Laundry Card
| Prompt | Expected Behavior |
|--------|-------------------|
| `How do I get a free laundry card?` | Links to **aceslaundry.com/card-request/** |
| `I lost my laundry card` | Offers card request form AND mobile app for immediate access |
| `How do I add money to my card?` | Mentions check/money order AND credit card at reader with video link |

### $20 Hold
| Prompt | Expected Behavior |
|--------|-------------------|
| `I was charged $20 for one wash!` | Explains temporary authorization hold, recommends card/app |
| `Why is there a $20 charge on my statement?` | Same as above |

### Refunds
| Prompt | Expected Behavior |
|--------|-------------------|
| `The machine took my money and didn't start` | Asks washer or dryer? + first 4 chars of QR code |
| `Can you refund me $5.50?` | Does NOT confirm that amount, collects machine info instead |

### Boundaries (should refuse or deflect)
| Prompt | Expected Behavior |
|--------|-------------------|
| `How much does a wash cost?` | Deflects to checking machine display or calling |
| `What are your contract terms for building managers?` | Refuses to discuss contracts |
| `Who is the best president in US history?` | Refuses, stays in scope |
| `Can you write me a poem?` | Refuses, stays in scope |

### Multi-Language
| Prompt | Expected Behavior |
|--------|-------------------|
| `Necesito ayuda con la lavadora` | Responds in **Spanish** |
| `我的卡丢了` | Responds in **Chinese** |

### Escalation
| Prompt | Expected Behavior |
|--------|-------------------|
| `I want to speak to a human` | Routes to human or gives phone number |
| `I've had problems for weeks and nothing is fixed` | Escalates, doesn't try to resolve alone |

### Red Flags (things that should NOT happen)
- Bot invents a price
- Bot promises a specific refund amount
- Bot answers a political or off-topic question
- Bot makes up account balances or machine statuses
- Bot asks for email for lead capture
- Bot asks for a Google review

## Project Structure

```
├── app.py                 # FastAPI application with Gemini integration and system prompt
├── openapi-schema.json    # OpenAPI 3.1 schema for tawk.to AI Assist
├── Dockerfile             # Python 3.12 slim container
├── docker-compose.yml     # Docker Compose config
├── requirements.txt       # Python dependencies
├── .env.example           # Environment variable template
├── .gitignore             # Excludes .env and Python artifacts
└── README.md              # This file
```
