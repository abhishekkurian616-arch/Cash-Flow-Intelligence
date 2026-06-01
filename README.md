## Summary

Most AI-powered financial tools send your business data to cloud servers — OpenAI, Google, or proprietary SaaS platforms. This project takes a different approach: **all AI inference runs locally on your own hardware**, with a fine-tuned model that deeply understands accounting vocabulary, transaction categorization, and cash flow dynamics.

The result is a standalone product that small businesses and accountancy firms can deploy on-premise, keeping sensitive financial data entirely within their own walls while still getting intelligent, automated insights.

---

## The Problem It Solves

Small business owners and bookkeepers spend significant time on tasks that are repetitive but require financial context:

- Manually categorizing bank transactions into accounting categories
- Building cash flow forecasts in spreadsheets by hand
- Monitoring accounts receivable aging and chasing overdue invoices
- Calculating burn rate and runway for planning purposes

Existing tools like Float, Pulse, and Spotlight Reporting solve parts of this — but they are cloud-dependent, expensive at scale, and rely on general-purpose AI that has no deep specialization in accounting.

This project builds a **domain-specialized local AI agent** that handles all of the above, integrated directly with QuickBooks Online and Xero via their APIs.

---

## How It Works — The AI Angle

The core AI component is a **fine-tuned large language model** running locally via Ollama. The fine-tuning process uses QLoRA (Quantized Low-Rank Adaptation), which allows a consumer-grade laptop to train a specialized version of an open-source model (such as Qwen2.5 7B) on a custom dataset of accounting instruction/response pairs.

### The AI Pipeline

```
QuickBooks / Xero API
        ↓
  FastAPI Service Layer
        ↓
  LangChain Agent Orchestration
        ↓
  Fine-tuned Local LLM (Ollama)
        ↓
  Vector Database (RAG — company-specific context)
        ↓
  Structured JSON Output → Dashboard + Alerts
```

### What Makes It "AI" Rather Than Just Automation

Traditional rule-based systems can categorize transactions using keyword matching. This agent goes further:

- It **reasons about ambiguous transactions** (e.g., "STRIPE PAYMENT $4,200" could be revenue or a software subscription — context determines which)
- It **generates narrative explanations** alongside structured data, not just labels
- It **forecasts probabilistically** by identifying patterns in historical data and producing scenario-based projections (best/base/worst case)
- It **learns from corrections** — when a user overrides a categorization, that correction can be added back to the training dataset and the model re-fine-tuned

---

## Technical Stack

### Infrastructure

| Layer | Technology | Purpose |
|---|---|---|
| Host OS | Elementary OS (Ubuntu 24.04 base) | Runs on old consumer laptop |
| Containerization | Docker Engine | Isolated, reproducible services |
| Container Management | Portainer | Browser-based Docker dashboard |
| Reverse Proxy | Caddy | Local HTTPS routing |

### AI & Model Layer

| Component | Technology | Purpose |
|---|---|---|
| Model Runtime | Ollama | Runs GGUF models locally |
| Base Model | Qwen2.5 7B (Q4 quantized) | Strong at reasoning and numbers |
| Fine-tuning | Unsloth + QLoRA | Efficient training on consumer hardware |
| Chat Interface | Open WebUI | Test and interact with models |
| Model Format | GGUF (via llama.cpp) | Optimized for CPU/low-VRAM inference |

### Agent & Application Layer

| Component | Technology | Purpose |
|---|---|---|
| Agent Orchestration | LangChain | Tool calling, memory, multi-step reasoning |
| Vector Database | Qdrant | RAG — retrieval of company-specific context |
| API Service | FastAPI (Python) | Exposes agent to webhooks and frontend |
| Database | PostgreSQL | Transaction history, forecasts, user data |
| Cache | Redis | Session state, fast lookups |

### Integrations

| Service | Method | Data Flow |
|---|---|---|
| QuickBooks Online | OAuth 2.0 + Webhooks | New transactions trigger AI categorization |
| Xero | OAuth 2.0 + Polling | Sync invoices, bills, bank feeds |
| Email Alerts | Resend API | Proactive cash flow warnings |

---

## Build Roadmap — Baby Steps

### Phase 1 — Infrastructure Foundation ✅
- Install Docker Engine on Elementary OS
- Set up Portainer for container management
- Configure Docker networking and storage paths
- Verify container runtime with hello-world test

### Phase 2 — Local LLM Stack ✅
- Deploy Ollama as a persistent Docker container
- Pull and test base models (llama3.2:3b, qwen2.5:7b)
- Set up Open WebUI as a chat interface
- Create a custom CashFlowBot Modelfile with a finance-focused system prompt

### Phase 3 — Fine-tuning Pipeline 🔜
- Build a training dataset of 500–2000 accounting instruction/response pairs
- Set up Unsloth in a Docker container for QLoRA fine-tuning
- Train a specialized version of Qwen2.5 7B on the cash flow dataset
- Convert to GGUF format and load back into Ollama
- Evaluate the fine-tuned model against the base model on finance tasks

### Phase 4 — Agent Stack 🔜
- Build a LangChain agent with tools for reading CSVs, calling APIs, and generating reports
- Set up Qdrant vector database for RAG (company transaction history as context)
- Wrap the agent in a FastAPI service layer
- Write webhook handlers for QuickBooks and Xero API events

### Phase 5 — Fintech Product 🔜
- Build a React + Recharts dashboard showing cash position and 90-day forecast
- Implement AR aging analysis and overdue invoice alerts
- Add scenario modeling (best/base/worst case cash flow projections)
- Package as a Docker Compose stack for on-premise deployment
- Design multi-tenant architecture for accountancy firm white-labeling

---

## Training Data — What the Model Learns

The fine-tuned model is trained on instruction/response pairs covering the core financial tasks. Each example follows the Alpaca format:

```json
{
  "instruction": "Categorize this transaction and suggest the QuickBooks account",
  "input": "STRIPE PAYMENT $4,200 — software subscription renewal",
  "output": {
    "category": "Software & SaaS",
    "qbo_account": "Computer and Internet Expenses",
    "type": "expense",
    "recurring": true,
    "confidence": "high"
  }
}
```

```json
{
  "instruction": "Calculate burn rate and runway from these figures",
  "input": "Starting balance: $45,000. Monthly expenses: $12,500. Monthly revenue: $8,200.",
  "output": "Net monthly burn: $4,300. Runway: 10.5 months at current burn rate. Recommendation: increase revenue or reduce costs to extend runway beyond 12 months."
}
```

Training categories include:
- Transaction categorization (expenses, revenue, transfers)
- Invoice aging classification and collection probability
- Cash flow forecasting (30, 60, 90 day horizons)
- Burn rate and runway calculation
- Recurring expense detection
- Anomaly flagging (unusual spend patterns)

---

## The Competitive Moat

| Feature | This Project | Float / Pulse | ChatGPT + Accountant |
|---|---|---|---|
| Data stays on-premise | ✅ | ❌ | ❌ |
| Fine-tuned on accounting | ✅ | N/A | ❌ |
| Works offline | ✅ | ❌ | ❌ |
| QBO + Xero native | ✅ | ✅ | ❌ |
| Customizable per client | ✅ | Limited | N/A |
| One-time hardware cost | ✅ | Per seat SaaS | Per token |

---

## Hardware Requirements

This project is deliberately designed to run on modest hardware — an old laptop nobody is using:

- **Minimum:** 8GB RAM, 4-core CPU, 256GB storage
- **Recommended:** 16GB RAM, 6+ core CPU, 512GB SSD
- **GPU optional:** NVIDIA with 4GB+ VRAM speeds up inference and fine-tuning significantly but is not required

The Qwen2.5 7B model at Q4 quantization runs at approximately 3–5 tokens per second on CPU-only hardware — slow by cloud standards, but entirely adequate for background processing of transactions and nightly forecast generation.

---

## Privacy & Ethics Considerations

- **No financial data ever leaves the machine** — all inference is local
- **Model outputs are advisory only** — the system never makes autonomous financial decisions
- **User corrections improve the model** — with explicit consent, corrections feed back into fine-tuning
- **Transparent confidence scores** — the model flags uncertainty rather than hallucinating figures
- **Audit trail** — all agent decisions are logged with the prompt, context, and response for review

---

## Current Status

| Milestone | Status |
|---|---|
| Docker Engine installed on Elementary OS | ✅ Complete |
| Portainer dashboard running | ✅ Complete |
| Ollama container with qwen2.5:7b and llama3.2:3b | ✅ Complete |
| Open WebUI connected and tested | ✅ Complete |
| CashFlowBot Modelfile | 🔜 Next session |
| Training dataset construction | 🔜 Upcoming |
| Fine-tuning pipeline | 🔜 Upcoming |
| QuickBooks API integration | 🔜 Upcoming |
| Dashboard frontend | 🔜 Upcoming |

---

## References & Inspiration

- [Elements of AI](https://www.elementsofai.com/) — course that sparked this project
- [Ollama](https://ollama.com/) — local model runtime
- [Unsloth](https://github.com/unslothai/unsloth) — efficient fine-tuning
- [Open WebUI](https://github.com/open-webui/open-webui) — local chat interface
- [LangChain](https://langchain.com/) — agent orchestration
- [QuickBooks Online API](https://developer.intuit.com/) — accounting integration
- [Xero API](https://developer.xero.com/) — accounting integration

---

*Built step by step on a Lenovo IdeaPad running Elementary OS — proving that meaningful AI projects don't require cloud compute or large budgets.*
