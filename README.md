# 🏠 Real Estate AI Agents — n8n

> **AI-powered real estate automation workflows built with n8n, RAG, Qdrant, OpenRouter, Vapi, Telegram, and guardrails.**

This repository contains a collection of **production-oriented n8n workflows for real estate AI assistants**.

The workflows are designed around a common principle:

**Retrieve trusted property information → validate the context → generate a grounded response → prevent unsupported answers.**

The system can be used for property information assistants, voice-based real estate agents, Telegram property assistants, and RAG-powered knowledge-base APIs.

---

## ✨ Features

* 🤖 AI-powered real estate assistants
* 🏠 Property knowledge-base Q&A
* 🔎 Retrieval-Augmented Generation (RAG)
* 🧠 Qdrant vector search
* 🔤 OpenRouter embeddings
* 💬 Telegram conversational assistant
* 📞 Vapi voice assistant integration
* 🛡️ Input and output guardrails
* 🚫 Protection against unsupported/hallucinated answers
* 📚 Context-aware property responses
* 🔄 Conversation/session handling
* 📊 Structured execution logging
* ⚡ n8n-based visual workflow automation
* 🔌 Webhook APIs for external applications

---

# 🏗️ Architecture

The project is built as a set of independent n8n workflows that share a common RAG architecture.

```text
                    ┌─────────────────────┐
                    │    User / Client    │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        ┌───────────┐    ┌───────────┐   ┌─────────────┐
        │  Telegram │    │   Vapi    │   │   Web/API   │
        └─────┬─────┘    └─────┬─────┘   └──────┬──────┘
              │                │                │
              └────────────────┼────────────────┘
                               ▼
                       ┌───────────────┐
                       │   n8n         │
                       │   Workflow    │
                       └───────┬───────┘
                               │
                               ▼
                       ┌───────────────┐
                       │ Input         │
                       │ Guardrails    │
                       └───────┬───────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Query Preparation /  │
                    │ Intent Validation    │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ OpenRouter Embedding │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │       Qdrant         │
                    │   Vector Search      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Context Processing / │
                    │ Relevance Checking   │
                    └──────────┬───────────┘
                               │
                         Enough Context?
                         /             \
                       No               Yes
                       │                 │
                       ▼                 ▼
                Safe Response      ┌─────────────┐
                                   │     LLM      │
                                   │   Response   │
                                   └──────┬──────┘
                                          │
                                          ▼
                                  ┌───────────────┐
                                  │ Output        │
                                  │ Guardrails    │
                                  └───────┬───────┘
                                          │
                                          ▼
                                  ┌───────────────┐
                                  │ Final Answer  │
                                  └───────────────┘
```

---

# 📂 Workflows

The repository currently contains three main workflows.

| Workflow                                         | Purpose                                    | Interface      |
| ------------------------------------------------ | ------------------------------------------ | -------------- |
| `Call Analysis.json`                             | Voice-based grounded real estate assistant | Vapi + Webhook |
| `RAG QA - OpenRouter + Qdrant + Guardrails.json` | Generic RAG question-answering API         | Webhook        |
| `Sunrise Residences Grounded Assistant (1).json` | Property-specific conversational assistant | Telegram       |

---

# 1. 📞 Call Analysis — Vapi Voice RAG

### File

```text
Call Analysis.json
```

This workflow provides a **voice-oriented real estate knowledge assistant** using Vapi as the external voice interface.

The workflow exposes the webhook:

```text
vapi-voice-rag
```

The incoming request is processed, validated, and converted into a clean query before retrieval.

The workflow extracts information such as:

* User query
* Transcript
* Call ID
* Caller ID
* Session ID
* Tool call ID
* Timestamp

It also removes common voice filler words such as:

```text
uh
um
erm
uhh
umm
```

before processing the query.

### Processing Flow

```text
Vapi
  ↓
Inbound Webhook
  ↓
Extract Call Data
  ↓
Validate Request
  ↓
Load Session
  ↓
Prepare Query
  ↓
Scope Check
  ↓
Generate Embedding
  ↓
Qdrant Vector Search
  ↓
Process Results
  ↓
Evidence Check
  ↓
Grounded LLM Response
  ↓
Optional Answer Validation
  ↓
Compose Response
  ↓
Respond to Vapi
  ↓
Save Session
  ↓
Structured Logging
```

The workflow explicitly separates retrieval from response generation and includes an evidence check before allowing the LLM to answer.

### RAG Configuration

The workflow uses:

```text
Vector Database:
Qdrant

Collection:
rag_documents

Embedding Provider:
OpenRouter

Embedding Model:
openai/text-embedding-3-small

LLM:
openai/gpt-4o-mini

Top K:
5

Similarity Threshold:
0.15

Minimum Results:
1
```

The workflow's system instructions require answers to use only supplied property evidence and provide a fallback when the information is unavailable.

---

# 2. 🧠 RAG QA — OpenRouter + Qdrant + Guardrails

### File

```text
RAG QA - OpenRouter + Qdrant + Guardrails.json
```

This workflow provides a reusable **RAG question-answering API**.

### Endpoint

```text
POST /webhook/rag-query
```

The workflow starts with an input guardrail before performing retrieval.

```text
Webhook
   ↓
Input Guardrail
   ↓
Input Allowed?
   │
   ├── No → Guardrail Response
   │
   └── Yes
          ↓
     Query Embedding
          ↓
     Qdrant Search
          ↓
     Relevance Check
          ↓
     Relevant Context?
       /          \
     No            Yes
     │              │
     ▼              ▼
No Context      DeepSeek
Response        via OpenRouter
                    │
                    ▼
              Output Guardrail
                    │
                    ▼
              Final Response
```

The workflow uses Qdrant for vector retrieval and DeepSeek through OpenRouter for final answer generation.

### LLM Configuration

```text
Provider:
OpenRouter

Model:
deepseek/deepseek-chat

Temperature:
0.1
```

The prompt instructs the model to answer only from retrieved context and treat retrieved documents as data rather than instructions.

### Output Guardrail

The workflow checks generated responses for suspicious content such as references to:

```text
system prompts
developer messages
API keys
secret keys
internal instructions
```

If the response triggers the guardrail, a safe fallback response is returned instead.

---

# 3. 💬 Sunrise Residences Grounded Assistant

### File

```text
Sunrise Residences Grounded Assistant (1).json
```

This workflow provides a **Telegram-based real estate assistant** for the Sunrise Residences property knowledge base.

### User Flow

```text
Telegram User
      ↓
Telegram Trigger
      ↓
Prepare Query
      ↓
Intent Check
      ↓
Parse Intent
      ↓
Is Query Valid?
    /       \
  No         Yes
  │           │
  ▼           ▼
Telegram    Embedding
Response       │
               ▼
          Qdrant Search
               │
               ▼
         Process Results
               │
               ▼
         Evidence Check
          /          \
        No            Yes
        │              │
        ▼              ▼
   Safe Reply       AI Agent
                         │
                         ▼
                   Telegram Reply
```

The workflow uses Telegram as the conversational interface, OpenRouter for embeddings, Qdrant for retrieval, and an AI Agent for grounded responses.

### Embedding

```text
Provider:
OpenRouter

Model:
openai/text-embedding-3-small
```

### Vector Search

The workflow queries the Qdrant `rag_documents` collection and retrieves up to five matching records with their payloads.

### AI Model

The workflow contains a DeepSeek chat model connected to the AI Agent and also uses DeepSeek for intent-related processing.

---

# 🔐 Grounded AI Architecture

A major focus of this project is **reducing hallucinations in real estate assistants**.

Instead of allowing an LLM to freely answer property questions:

```text
User Question
     ↓
Vector Retrieval
     ↓
Relevant Property Data
     ↓
Context Validation
     ↓
LLM
     ↓
Guardrails
     ↓
Answer
```

The LLM receives retrieved property information and is instructed not to invent facts.

For example, if the knowledge base contains:

```text
2 BHK apartments
Area: 1,250 sq ft
```

A user asking:

> What is the apartment size?

can receive an answer based on the retrieved information.

But if the user asks:

> Do you have a 4 BHK available?

and the knowledge base does not contain that information, the workflow can return a safe "information not available" response rather than inventing availability.

---

# 🛡️ Guardrails

The workflows implement multiple layers of protection.

## Input Guardrails

Input validation occurs before expensive retrieval and generation steps.

Possible invalid or unsupported requests can be rejected before reaching the LLM.

## Retrieval Validation

Retrieved documents are checked for relevance before they are passed to the model.

## Evidence Threshold

The workflows contain explicit logic for determining whether enough evidence exists to answer a question.

```text
Question
   ↓
Retrieve Documents
   ↓
Rank / Process
   ↓
Enough Evidence?
   ├── No → Safe Response
   └── Yes → Generate Answer
```

## Output Guardrails

Generated responses are inspected before being returned.

The RAG QA workflow, for example, blocks suspicious references to internal prompts, API keys, secrets, and internal instructions.

---

# 🧰 Technology Stack

| Technology                 | Purpose                                          |
| -------------------------- | ------------------------------------------------ |
| **n8n**                    | Workflow automation and orchestration            |
| **Qdrant**                 | Vector database and semantic search              |
| **OpenRouter**             | LLM and embedding API gateway                    |
| **DeepSeek**               | Grounded response generation / intent processing |
| **GPT-4o-mini**            | Voice RAG response generation in Call Analysis   |
| **text-embedding-3-small** | Query/document embeddings                        |
| **Vapi**                   | Voice AI interface                               |
| **Telegram**               | Conversational interface                         |
| **Webhooks**               | API integration                                  |
| **JavaScript**             | n8n Code nodes and data processing               |

---

# 🚀 Getting Started

## Prerequisites

You need:

* n8n
* Qdrant instance
* OpenRouter API key
* Telegram Bot Token — for the Telegram workflow
* Vapi account — for the voice workflow
* A populated Qdrant collection

---

## 1. Clone the Repository

```bash
git clone https://github.com/Pawan1724/Realestate-Agents-n8n.git

cd Realestate-Agents-n8n
```

---

## 2. Start n8n

If you already have n8n installed, start your instance normally.

For Docker:

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  n8nio/n8n
```

For a persistent deployment, use Docker Compose and a persistent volume.

---

# 3. Import the Workflows

Open n8n:

```text
http://localhost:5678
```

Then:

```text
Workflows
   ↓
Import from File
   ↓
Select JSON workflow
```

Import:

```text
Call Analysis.json
RAG QA - OpenRouter + Qdrant + Guardrails.json
Sunrise Residences Grounded Assistant (1).json
```

---

# 🔑 Credentials & Configuration

After importing the workflows, configure the required credentials inside n8n.

### OpenRouter

Used for:

* LLM generation
* Query embeddings

Configure your OpenRouter authentication in n8n rather than committing API keys into workflow JSON.

### Qdrant

Configure:

```text
QDRANT_URL
QDRANT_API_KEY
QDRANT_COLLECTION
```

The current workflows use:

```text
rag_documents
```

as the vector collection.

### Telegram

For the Sunrise Residences workflow, configure:

```text
Telegram Bot Token
```

and attach the credential to the Telegram Trigger and Telegram response nodes.

### Vapi

For the voice workflow, configure the Vapi assistant/tool to call the n8n webhook:

```text
vapi-voice-rag
```

The workflow expects Vapi-compatible request data and returns the generated answer through the webhook response.

---

# 📚 Preparing the Knowledge Base

The RAG workflows expect property information to already exist in Qdrant.

A typical document can contain information such as:

```text
Property Name:
Sunrise Residences

Location:
Hyderabad

Property Type:
Residential Apartments

Configurations:
2 BHK, 3 BHK

Amenities:
Swimming Pool
Gym
Parking
Security

Pricing:
...

Availability:
...

Contact:
...
```

Documents should be converted into embeddings and stored in the Qdrant collection:

```text
rag_documents
```

Each vector should retain useful payload metadata such as:

```json
{
  "text": "Property information...",
  "property": "Sunrise Residences",
  "location": "Hyderabad",
  "document_type": "property_details"
}
```

---

# 🔄 RAG Pipeline

The complete RAG process is:

### Step 1 — Receive Question

```text
"What is the price of the 2 BHK?"
```

### Step 2 — Validate Input

The workflow checks whether the request is valid and within the supported scope.

### Step 3 — Create Embedding

The question is converted into a vector using:

```text
openai/text-embedding-3-small
```

through OpenRouter.

### Step 4 — Search Qdrant

The vector is searched against:

```text
rag_documents
```

### Step 5 — Process Results

Retrieved documents are filtered, ranked, and checked for relevance/evidence.

### Step 6 — Generate Answer

The LLM receives:

```text
User Question
+
Retrieved Context
```

and generates a grounded answer.

### Step 7 — Output Validation

The answer passes through output guardrails.

### Step 8 — Return Response

The final response is returned to:

```text
Telegram
Vapi
API Client
```

depending on the workflow.

---

# 📡 API Example

The RAG QA workflow exposes:

```http
POST /webhook/rag-query
```

Example request:

```json
{
  "question": "What amenities are available at Sunrise Residences?"
}
```

Example response:

```json
{
  "ok": true,
  "grounded": true,
  "answer": "Sunrise Residences offers amenities including ...",
  "model": "deepseek/deepseek-chat"
}
```

If sufficient context is not available:

```json
{
  "ok": true,
  "grounded": false,
  "answer": "The requested information is not available in the knowledge base.",
  "retrieved_count": 0
}
```

---

# 📞 Vapi Integration

The Call Analysis workflow exposes a webhook for Vapi:

```text
POST /webhook/vapi-voice-rag
```

The workflow can process:

* Voice transcript
* Tool calls
* Caller information
* Call/session identifiers
* User questions

The response is formatted for Vapi's tool-call response structure.

This allows the architecture to support:

```text
Customer
   ↓
Phone Call
   ↓
Vapi
   ↓
n8n
   ↓
Qdrant
   ↓
LLM
   ↓
n8n
   ↓
Vapi
   ↓
Customer
```

---

# 💬 Telegram Integration

The Sunrise Residences assistant uses Telegram as the front-end.

```text
Telegram User
      ↓
Telegram Trigger
      ↓
n8n
      ↓
Intent Detection
      ↓
RAG
      ↓
AI Agent
      ↓
Telegram
```

This makes it possible to build a lightweight property-information assistant without developing a separate chat frontend.

---

# ⚙️ Configuration

Important values used by the workflows include:

```text
Qdrant collection:
rag_documents

Embedding model:
openai/text-embedding-3-small

RAG QA LLM:
deepseek/deepseek-chat

RAG QA temperature:
0.1

Call Analysis LLM:
openai/gpt-4o-mini

Top K:
5

Minimum results:
1
```

The Call Analysis workflow also limits voice responses to a short format suitable for phone conversations.

---

# 🔒 Security

### Never commit API keys

Do not place credentials directly inside workflow JSON files.

Use:

* n8n Credentials
* Environment variables
* Secret management
* Restricted API keys

### Recommended Production Practices

* Restrict Qdrant network access
* Use HTTPS for webhooks
* Authenticate external webhook requests
* Rotate API keys periodically
* Apply rate limits
* Log failed requests
* Avoid storing unnecessary caller information
* Separate development and production credentials

---

# 🧪 Testing

Before activating a workflow, test each layer independently.

### RAG Test

```text
Question
   ↓
Embedding
   ↓
Qdrant
   ↓
Retrieved Context
   ↓
LLM
   ↓
Guardrail
```

### Test Questions

Use questions that are definitely present in the knowledge base:

```text
What property configurations are available?

What amenities are available?

Where is the property located?

What is the apartment size?
```

Also test unsupported questions:

```text
What will the property price be next year?

Can you guarantee availability?

What is the exact future appreciation?
```

The assistant should not invent answers when the required information is missing.

---

# 📈 Production Improvements

The current workflows provide a strong foundation for a production real estate AI platform.

Recommended next steps:

### 1. Property CRM Integration

Connect:

```text
n8n
 ↓
CRM
 ↓
Lead
 ↓
Agent Assignment
```

### 2. Lead Qualification

Automatically extract:

```text
Name
Phone
Budget
Location
Property Type
Configuration
Purchase Timeline
Intent
```

### 3. Automated Follow-ups

Integrate:

```text
WhatsApp
Email
SMS
Telegram
```

for lead follow-up.

### 4. Property Availability

Connect the RAG system to a live property database so availability is retrieved from current transactional data instead of static documents.

### 5. Agent Handoff

If the AI cannot answer or the customer wants human assistance:

```text
AI Assistant
      ↓
Lead Qualification
      ↓
Human Agent
```

### 6. Analytics Dashboard

Track:

```text
Total Conversations
Qualified Leads
Property Queries
Failed Queries
Human Handoffs
Conversion Rate
Average Response Time
```

### 7. Observability

Add structured logging for:

```text
request_id
session_id
latency
retrieval_count
similarity_score
model
token_usage
guardrail_status
error
```

---

# 🗂️ Repository Structure

```text
Realestate-Agents-n8n/
│
├── Call Analysis.json
│
├── RAG QA - OpenRouter + Qdrant + Guardrails.json
│
├── Sunrise Residences Grounded Assistant (1).json
│
└── README.md
```

---

# 🎯 Use Cases

This project can be adapted for:

* Real estate property assistants
* Property listing Q&A
* Real estate lead qualification
* Voice-based property assistants
* Telegram property bots
* Website chatbots
* Property knowledge bases
* Real estate sales automation
* Customer support automation
* RAG-based property search
* AI-assisted real estate operations

---

# 🔮 Future Architecture

The workflows can eventually be combined into a complete real estate AI platform:

```text
                         ┌──────────────────┐
                         │  Property / CRM   │
                         │     Database      │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ Knowledge / RAG   │
                         │     Pipeline      │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │      Qdrant      │
                         └────────┬─────────┘
                                  │
                ┌─────────────────┼─────────────────┐
                │                 │                 │
                ▼                 ▼                 ▼
           ┌─────────┐       ┌─────────┐      ┌─────────┐
           │ Telegram│       │  Vapi   │      │ Website │
           └────┬────┘       └────┬────┘      └────┬────┘
                │                 │                 │
                └─────────────────┼─────────────────┘
                                  ▼
                            ┌─────────────┐
                            │     n8n     │
                            │ AI Workflow │
                            └──────┬──────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
                    ▼              ▼              ▼
                 Lead CRM      Follow-up      Analytics
```

---

# 🤝 Contributing

Contributions and improvements are welcome.

Suggested contributions:

* New real estate workflows
* CRM integrations
* WhatsApp automation
* Lead qualification
* Property search
* Better retrieval strategies
* Additional guardrails
* Monitoring and observability
* Production deployment examples

---

# 👨‍💻 Author

**Pawan Kumar Salikanti**

AI/ML Engineer focused on:

* Artificial Intelligence
* Generative AI
* RAG
* AI Agents
* n8n Automation
* Computer Vision
* NLP
* Backend & API Development

---

## ⭐ Project

If you find this project useful, consider starring the repository and contributing improvements.

**Repository:**
https://github.com/Pawan1724/Realestate-Agents-n8n
