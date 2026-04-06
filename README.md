# Tawk.to AI Support Agent

AI-powered customer support agent that integrates with tawk.to AI Assist using Google Gemini.

## Setup

1. Copy `.env.example` to `.env` and fill in your values
2. Run with Docker: `docker-compose up --build`
3. Or run locally: `pip install -r requirements.txt && uvicorn app:app --port 8080`

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `GEMINI_API_KEY` | Google Gemini API key | Yes |
| `GEMINI_MODEL` | Gemini model name (default: `gemini-2.0-flash`) | No |
| `API_SECRET_KEY` | Secret key for API authentication | Yes |

## Endpoints

- `GET /health` - Health check
- `POST /chat` - Send a message (requires `X-API-Key` header)

## tawk.to Integration

1. Upload `openapi-schema.json` to tawk.to AI Assist
2. Set the API Base URL to your deployment URL
3. Configure API Key authentication with header `X-API-Key`
