# Peloton AI Agent Platform

A multi-agent AI system built with **LangChain + LangGraph + GPT-4o** that automates customer service workflows for Peloton's five core business domains. A router agent classifies incoming queries and dispatches them to the right specialist agent, each of which uses domain-specific tools to resolve the request end-to-end.

---

## Architecture

```
User Query
    │
    ▼
┌─────────────┐
│ Router Agent│  ← Classifies query into one of 5 domains
└──────┬──────┘
       │
  ┌────┴─────────────────────────────────────────┐
  │                                              │
  ▼            ▼            ▼          ▼         ▼
Business/   Data         Member/    Order/   Product
Marketing   Science      Fraud      Shipping  Recommendation
 Agent       Agent        Agent      Agent     Agent
```

Built on **LangGraph's StateGraph** — each node is a specialist agent with its own tools, and the router drives conditional edges between them. State is typed with `TypedDict` and flows cleanly through the graph.

---

## Agents & User Stories

### 🏷️ Business / Marketing
| Story | Task |
|-------|------|
| US1 | Analyze Black Friday Campaign ROI — identify top-performing ads and recommend budget reallocation |
| US2 | Q2 Content Strategy — surface high-engagement themes and propose a social calendar |
| US3 | Competitor Analysis — compare Peloton membership growth against NordicTrack and Echelon |

**Tools:** `get_campaign_analytics`, `get_content_engagement`, `get_competitor_data`

---

### 📊 Data Science
| Story | Task |
|-------|------|
| US1 | Cycling Performance — review last 30 classes and suggest output/cadence improvements |
| US2 | Workout Consistency — show 3-month patterns and recommend a personalized schedule |
| US3 | Demographic Report — popular class types by age group and instructor completion rates |

**Tools:** `get_cycling_performance`, `get_workout_consistency`, `get_demographic_report`

---

### 🔐 Membership / Fraud Detection
| Story | Task |
|-------|------|
| US1 | Suspicious Device Activity — list recent sign-ins and revoke unrecognized devices |
| US2 | Password Reset — guide locked-out members through OTP-based account recovery |
| US3 | Membership Tier Comparison — compare All-Access vs App+ and recommend the right plan |

**Tools:** `list_recent_devices`, `revoke_device`, `initiate_password_reset`, `get_membership_tiers`

---

### 📦 Order / Shipping
| Story | Task |
|-------|------|
| US1 | Order Tracking — look up shipment status, carrier, and ETA |
| US2 | Return Process — initiate returns for damaged/defective items |
| US3 | Delayed Order Escalation — flag and escalate orders that have missed their ship date |

**Tools:** `track_order`, `initiate_return`, `escalate_delayed_order`

---

### 🛍️ Product Recommendation
| Story | Task |
|-------|------|
| US1 | Yoga Accessories — recommend mats and blocks based on recent class history |
| US2 | Bike vs Bike+ — feature-by-feature comparison with a budget-aware recommendation |
| US3 | Tread Cross-Sell — suggest accessory bundles for customers who just bought a Tread |

**Tools:** `get_member_class_history`, `get_product_catalog`, `compare_products`, `get_cross_sell_bundles`

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| LLM | GPT-4o via `langchain-openai` |
| Agent Framework | LangChain 1.0 |
| Graph Orchestration | LangGraph `StateGraph` |
| Training Data | Custom JSON few-shot pairs (train/test split per agent) |
| Notebook | Jupyter (Python 3) |

---

## Repo Structure

```
├── notebook/
│   └── peloton_ai_agents.ipynb   # Full implementation — all 15 user stories
└── training_data/
    ├── business_marketing_training.json
    ├── business_marketing_testing.json
    ├── data_science_training.json
    ├── data_science_testing.json
    ├── membership_fraud_training.json
    ├── membership_fraud_testing.json
    ├── order_shipping_training.json
    ├── order_shipping_testing.json
    ├── product_recommendation_training.json
    └── product_recommendation_testing.json
```

---

## How to Run

1. **Clone the repo and install dependencies**
   ```bash
   pip install langchain langchain-openai langgraph openai python-dotenv
   ```

2. **Set your OpenAI API key**
   ```bash
   export OPENAI_API_KEY="sk-..."
   ```
   Or create a `.env` file:
   ```
   OPENAI_API_KEY=sk-...
   ```

3. **Open the notebook**
   ```bash
   jupyter notebook notebook/peloton_ai_agents.ipynb
   ```

4. **Run all cells** — Section 8 executes all 15 user stories end-to-end, and Section 9 runs the router regression tests against the held-out testing data.

---

## Few-Shot Training

Each agent is bootstrapped with domain-specific few-shot examples loaded from the `training_data/` folder. The `load_training_pairs()` function pulls up to 4 examples per agent and injects them into the system prompt before the agent handles live queries. Separate testing files are used for router regression tests to avoid data leakage.

---

## Notes

- All tool responses use mock data designed to simulate realistic Peloton backend payloads
- The `human_review` flag in `AgentState` is set automatically for sensitive operations (device revocation, escalations)
- The graph is renderable as a Mermaid diagram via `app.get_graph().draw_mermaid_png()`
