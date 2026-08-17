# Customer Retention Intelligence Agent

> **A multi-agent AI system for proactive customer churn detection and Next Best Action (NBA) recommendations.**

**AI Proficient Capstone Project — Q2 2026**  
**Author:** Anantakumaar Raviprasath

---

## Project Overview

The **Customer Retention Intelligence Agent** is a multi-agent AI solution designed to proactively identify customers who may be at risk of churn and recommend an appropriate **Next Best Action (NBA)** before the customer leaves.

The system combines:

- **LangGraph** for multi-agent orchestration
- **ChromaDB + Retrieval-Augmented Generation (RAG)** for retention playbook retrieval
- **Azure OpenAI GPT-4o** for final response generation
- **Deterministic business rules** for risk-segment classification
- **CLV-tier-based personalization** for differentiated retention actions

The architecture is intentionally designed so that critical customer-risk classification is deterministic and auditable, while generative AI is used for the final narrative and recommendation output.

---

## Business Problem

Customer churn can be difficult to detect because disengagement often happens gradually. Traditional CRM workflows may identify a customer only after the customer has already decided to leave.

Common challenges addressed by this project include:

- **Silent churn:** declining app usage and email engagement without an explicit complaint
- **Reactive detection:** identifying risk only after significant disengagement
- **Inconsistent treatment:** different retention actions across agents, regions, or shifts
- **Limited scalability:** manually assessing large customer populations

The proposed system acts as a proactive intelligence layer that can identify at-risk customers, retrieve the relevant retention playbook, and generate a structured NBA recommendation.

---

## Solution

The system uses a **Centralized Supervisor Pattern** in LangGraph.

A Supervisor Agent coordinates three specialized agents in a mandatory sequence:

1. **Customer Profiler**
   - Retrieves the customer's behavioral profile.
   - Uses signals including recency, app sessions, email engagement, NPS, support tickets, and customer lifetime value (CLV).

2. **Churn Diagnostician**
   - Applies deterministic, priority-ordered business rules.
   - Classifies the customer into one of four predefined risk segments.

3. **Retention Strategist**
   - Uses RAG with ChromaDB to retrieve the most relevant retention playbook.
   - Adapts the recommended treatment based on the customer's risk segment and CLV tier.

4. **Response Generator**
   - Uses Azure OpenAI GPT-4o to generate the final structured retention response.
   - The LLM does **not** determine the risk segment or the underlying retention action.

---

## Architecture

```text
                         ┌─────────────────────────┐
                         │    Supervisor Agent     │
                         │       LangGraph         │
                         │  Orchestrates Workflow  │
                         └────────────┬────────────┘
                                      │
             ┌────────────────────────┼────────────────────────┐
             ▼                        ▼                        ▼
┌─────────────────────┐   ┌─────────────────────┐   ┌──────────────────────┐
│ Customer Profiler   │   │ Churn Diagnostician  │   │ Retention Strategist │
│                     │   │                     │   │                      │
│ fetch_customer_     │   │ classify_risk_      │   │ recommend_retention_ │
│ record              │   │ segment             │   │ action               │
└──────────┬──────────┘   └──────────┬──────────┘   └──────────┬───────────┘
           │                         │                         │
           │                         │                    ChromaDB RAG
           │                         │                         │
           └─────────────────────────┼─────────────────────────┘
                                     ▼
                         ┌─────────────────────────┐
                         │   Response Generator   │
                         │    Azure OpenAI GPT-4o │
                         └────────────┬────────────┘
                                      ▼
                         ┌─────────────────────────┐
                         │    Next Best Action     │
                         │  Segment + Decision +   │
                         │  Rationale + Treatment │
                         └─────────────────────────┘
```

---

## Risk Segmentation

Risk classification is intentionally handled through deterministic rules rather than an LLM. This provides consistency, transparency, and auditability.

| Priority | Risk Segment | NBA Decision | Key Signals |
|---|---|---|---|
| P1 | Support-Fatigued | ESCALATE | Support tickets ≥ 3, NPS ≤ 4, email open rate < 20% |
| P2 | Competitor-Switched | NURTURE | Recency > 90 days, 0 app sessions/month, email open rate < 10% |
| P3 | Disengaged | NURTURE | Recency > 60 days, app sessions < 2/month, email open rate < 15% |
| P4 | Healthy | HOLD | Recency ≤ 30 days, NPS ≥ 7 |

### Why deterministic classification?

Retention decisions require repeatability and governance. With deterministic rules, the same customer signals produce the same risk classification, making the decision easier to validate and audit.

---

## RAG with ChromaDB

Retention playbooks are stored separately from the application logic.

The RAG workflow is:

```text
Risk Segment
     │
     ▼
Embedding Query
(text-embedding-ada-002)
     │
     ▼
ChromaDB
Semantic Search
     │
     ▼
Top Matching Playbook
     │
     ▼
CLV-Tier Adaptation
     │
     ▼
NBA Report
```

### Why RAG?

Using RAG instead of hardcoded playbook lookups allows:

- Retention playbooks to be updated without changing application code
- Semantic retrieval instead of brittle string matching
- Business teams to maintain the knowledge base independently
- New or rephrased segment descriptions to be handled more flexibly

The LLM is used for narrative generation; it does not decide the customer's risk segment or underlying retention action.

---

## Example Decision Flow

For a high-risk customer, the system:

1. Retrieves the customer profile.
2. Evaluates the predefined risk rules.
3. Identifies the highest-priority matching risk segment.
4. Retrieves the corresponding retention playbook through ChromaDB.
5. Applies CLV-tier-specific treatment.
6. Generates a structured NBA report.

The final response contains:

- Customer risk segment
- Next Best Action
- Reasoning/rationale
- Recommended retention treatment
- Engagement policy

---

## Evaluation & Results

The solution was evaluated using two complementary approaches.

### 1. Deterministic Accuracy Validation

Agent outputs were compared directly against expert reference answers.

| Metric | Result |
|---|---:|
| Risk Segment Accuracy | **10/10 (100%)** |
| Decision Accuracy | **10/10 (100%)** |

### 2. LLM-as-a-Judge Evaluation

GPT-4o evaluated five dimensions for each validation customer:

| Dimension | Score |
|---|---:|
| Segment Accuracy | 5.0/5 |
| Decision Accuracy | 5.0/5 |
| Rationale Quality | 5.0/5 |
| Treatment Completeness | 4.2/5 |
| Engagement Policy | 5.0/5 |
| **Overall Average** | **4.84/5** |
| **Validation Pass Rate** | **10/10** |

**Key finding:** Treatment Completeness was the only dimension below 5.0. The evaluation found no wrong risk segments, incorrect engagement policies, or structural errors across the validation set.

---

## Business Recommendations

The proposed solution can be operationalized as a **nightly automated pre-triage layer** within an existing CRM workflow.

### ESCALATE

- Create a high-priority retention case
- Route the customer to a retention specialist
- Initiate personalized outreach within 24 hours
- Avoid promotional content until the service issue is resolved

### NURTURE

- Launch targeted win-back engagement
- Provide loyalty incentives and personalized recommendations
- Monitor engagement over the win-back period
- Close the win-back window after prolonged inactivity

### HOLD

- Continue standard engagement
- Avoid unnecessary retention spending
- Consider referral or advocacy programs for highly satisfied customers
- Continue passive monitoring and re-classify if risk signals change

---

## Technology Stack

| Technology | Purpose |
|---|---|
| Python | Core implementation |
| LangGraph | Multi-agent workflow orchestration |
| LangChain | LLM, tools, prompts and application framework |
| ChromaDB | Vector database for RAG |
| Azure OpenAI GPT-4o | Final response generation |
| Azure OpenAI Embeddings | Playbook embeddings |
| Pydantic | Structured data modelling |
| Pandas | Data processing and evaluation |
| Jupyter Notebook | Development and demonstration environment |

---

## Project Structure

A recommended GitHub repository structure is:

```text
customer-retention-intelligence-agent/
│
├── README.md
│
├── code/
│   └── customer_retention_intelligence_agent.ipynb
│
├── data/
│   ├── validation_customer_records.json
│   ├── test_customer_records.json
│   ├── risk_segments.json
│   └── retention_playbooks.json
│
├── results/
│   └── submission.csv
│
├── documentation/
│   ├── business-brief.pdf
│   └── capstone-presentation.pdf
│
├── requirements.txt
├── .gitignore
└── .env.example
```

> **Important:** API credentials and environment files containing secrets should never be committed to GitHub.

---

## Environment Setup

### Install dependencies

The original notebook installs the following packages:

```bash
pip install langchain==0.3.25
pip install langchain-core
pip install langchain-community
pip install langchain-openai
pip install langchain-text-splitters
pip install langchain-chroma==0.2.4
pip install python-dotenv
pip install langgraph==0.3.31
pip install pandas
pip install chromadb
```

### Configure Azure OpenAI

The notebook loads credentials from:

```text
retention_agent.env
```

The environment configuration includes values for:

```text
MODEL_ENDPOINT
CHAT_MODEL_NAME
AZURE_OPENAI_API_KEY
api_version
EMBEDDING_MODEL_NAME
MODEL_ENDPOINT_EMBEDDING
api_version_embedding
```

**Do not upload the actual `.env` / environment file or API key to GitHub.**

Use a local environment file or another secure secret-management mechanism when running the notebook.

---

## Input Data

The notebook expects the following input files:

```text
validation_customer_records.json
test_customer_records.json
risk_segments.json
retention_playbooks.json
```

The customer data is used to create a unified customer pool for the Customer Profiler. Risk segment definitions are loaded as priority-ordered deterministic rules, while retention playbooks are transformed into documents and indexed in ChromaDB.

---

## Output

The system generates structured NBA responses for customers and exports the final test results to:

```text
submission.csv
```

The submission contains:

```text
customer_id
agent_response
```

---

## Limitations & Future Improvements

The current solution has several limitations identified during the project:

- Risk thresholds are based on predefined business rules and may require periodic review as customer behavior changes.
- Additional customer attributes could improve personalization but are not currently available in the dataset.
- Customers who do not clearly match an existing risk segment may require manual review.
- For enterprise-scale deployment, the local vector database should be replaced with a managed production-grade vector search platform.
- The current implementation is a capstone/prototype and would require additional production engineering, governance, security, monitoring, and integration work before enterprise deployment.

---

## Key Takeaway

The project demonstrates how **business rules, retrieval-based knowledge, and generative AI can work together rather than relying on an LLM to make every decision**.

The architecture prioritizes:

**Auditability → Consistency → Explainability → Personalization → Scalability**

The central principle is:

> **AI in CRM does not have to be a black box. Every decision should be traceable to the signals that triggered it, the playbook it came from, and the action owner responsible for the next step.**

---

## Project Documentation

The repository can include:

- 📘 Business Brief
- 📊 Capstone Presentation
- 💻 Jupyter Notebook
- 📄 Sample/approved project data
- 📈 Evaluation results
- 🧾 Submission output

---

## Author

**Anantakumaar Raviprasath**

AI Proficient Training Program — Q2 2026  
Dentsu Global Services × Analytics Vidhya

---

## Disclaimer

This project was developed as an **AI Proficient Capstone Project** for learning and demonstration purposes. It is not presented as a production-ready customer-retention system.

Any production implementation should use appropriately governed, anonymized or authorized data and secure handling of credentials and business-sensitive information.
