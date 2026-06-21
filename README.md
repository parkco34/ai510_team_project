# Alcalá Voyages — AI Travel Planning Agent

**AAI-510: Agentic AI Systems | University of San Diego | June 2026**

**Team 6:**
- Cory Parker — Product Manager
- Akua Duffour — Data Engineer
- Paola Marsal — AI Engineer

---

## Project Overview

Alcalá Voyages is a San Diego-based European travel agency building a proprietary AI travel planning agent to assist customers with end-to-end trip planning through a conversational interface. The agent recommends European destinations, provides climate information, and suggests hotels grounded in real guest review data.

This project was built as part of the AAI-510 Agentic AI Systems capstone at the University of San Diego. It demonstrates a production-grade agentic system using the ReAct (Reasoning + Acting) architecture pattern, evaluated across two LLMs with an LLM-as-judge framework.

---

## Architecture

The agent follows the **ReAct (Reasoning + Acting)** pattern — the LLM alternates between reasoning steps and tool invocations to arrive at a final response.

```
User Query
    ↓
LLM Reasoning (GPT-4o or Claude Sonnet 4)
    ↓
Tool Registry
    ├── search_destinations()   → European cities dataset
    ├── get_climate_info()      → Monthly temperature data
    ├── recommend_hotels()      → TF-IDF RAG on hotel reviews
    └── reject_out_of_scope()  → Graceful rejection handler
    ↓
Response Synthesis
    ↓
MLflow Trace + LLM-as-Judge Evaluation
```

---

## Datasets

| Dataset | Source | Size | Use |
|---------|--------|------|-----|
| Worldwide Travel Cities | [Kaggle](https://www.kaggle.com/datasets/furkanima/worldwide-travel-cities-ratings-and-climate) | 560 cities, 167 countries | Destination search + climate tools |
| 515K Hotel Reviews Europe | [Kaggle](https://www.kaggle.com/datasets/jiashenliu/515k-hotel-reviews-data-in-europe) | 515,738 reviews, 1,492 hotels | Hotel recommendation via TF-IDF RAG |

Both datasets are loaded into Databricks Unity Catalog under `main.alcala_voyages`.

---

## Notebooks

| Notebook | Owner | Description |
|----------|-------|-------------|
| `Team6_510_Final_Project.ipynb` | Akua Duffour (DE) | Data pipeline — ingestion, cleaning, aggregation, TF-IDF index |
| `Team6_510_Agent_Notebook.ipynb` | Paola Marsal (AIE) | Agent definition, tools, ReAct loop, evaluation traces |

---

## LLMs Compared

| LLM | Provider | Cost per Query | Avg Judge Score |
|-----|----------|---------------|-----------------|
| GPT-4o | OpenAI | ~$0.005 | 3.80/5 |
| Claude Sonnet 4 | Anthropic | ~$0.003 | 4.07/5 |

**Result:** Claude Sonnet 4 outperformed GPT-4o by 0.27 points at approximately 40% lower cost. Claude Sonnet 4 is the recommended LLM for production deployment.

---

## Evaluation

Five traces were captured using MLflow and evaluated with an LLM-as-judge framework. Both GPT-4o and Claude Sonnet 4 were run on all 5 traces.

| Trace | Query | GPT-4o | Claude Sonnet 4 |
|-------|-------|--------|----------------|
| 1 | Cultural city in Spain (mid-range budget) | 5.0 | 5.0 |
| 2 | Climate in Paris in July | 5.0 | 5.0 |
| 3 | Romantic hotel in London | 4.33 | 4.67 |
| 4 | Flight booking — out-of-scope rejection | 2.33* | 2.33* |
| 5 | Adventure in Austria | 2.33 | 3.33 |
| **Overall** | | **3.80** | **4.07** |

*Trace 4: Low score reflects the judge evaluating from a user helpfulness perspective rather than agent correctness. Human review confirms the out-of-scope rejection was appropriate.

**MLflow Experiment:** `/Shared/alcala_voyages_agent_team6`

---

## Key Design Decisions

**TF-IDF over Databricks Vector Search**
Databricks Vector Search is not available on the free-tier workspace. We implemented TF-IDF cosine similarity search as the retrieval mechanism, achieving the same goal of semantic hotel retrieval from guest review text using scikit-learn.

**Europe-only hotel scope**
The 515K hotel reviews dataset covers 6 European countries (UK, Spain, France, Netherlands, Italy, Austria). Alcalá Voyages is positioned as a European travel specialist. Expanding hotel coverage is a one-dataset change that does not require architectural modification.

**ReAct over other patterns**
ReAct naturally supports chained tool calls — the agent reasons about destination, calls `search_destinations()`, reasons about climate, calls `get_climate_info()`, reasons about hotels, calls `recommend_hotels()`, then synthesizes a complete itinerary response.

---

## How to Run

### Prerequisites
- Databricks workspace (free tier supported)
- Anthropic API key (stored in Databricks secrets under scope `aai510`, key `anthropic_api_key`)
- OpenAI API key

### Step 1 — Run the Data Pipeline
Open `Team6_510_Final_Project.ipynb` in Databricks and run all cells. This loads both datasets into Unity Catalog, aggregates hotel reviews, and builds the TF-IDF index.

### Step 2 — Run the Agent
Open `Team6_510_Agent_Notebook.ipynb` in Databricks. Set your OpenAI API key in Cell 2, then run all cells. The agent will execute 5 evaluation traces and log results to MLflow.

---

## ROI Summary

| Metric | GPT-4o | Claude Sonnet 4 |
|--------|--------|----------------|
| Cost per query | ~$0.005 | ~$0.003 |
| Overall avg score | 3.80/5 | 4.07/5 |
| Performance delta | baseline | +0.27 (better) |
| Cost delta | baseline | ~40% cheaper |

**Recommendation:** Deploy Claude Sonnet 4 as the primary LLM. It delivers better performance at lower cost with no quality tradeoff for standard travel planning queries.

---

## AI Disclosure

AI tools including Claude (Anthropic) and GitHub Copilot were used for coding assistance, debugging support, and writing guidance. All agent design, tool implementation, evaluation logic, architectural decisions, and conclusions are the authors' own work. All AI usage has been reviewed, understood, and modified by the team.

---

## Repository Structure

```
ai510_team_project/
├── README.md
├── Team6_510_Final_Project.ipynb    # Data pipeline notebook (DE)
├── Team6_510_Agent_Notebook.ipynb   # Agent notebook (AIE)
└── .gitignore
```
