# Peloton AI Agent Platform

A graph-based customer service automation system for Peloton, built with LangChain, LangGraph, and GPT-4o. Incoming queries are classified by a router and handed off to one of five specialist agents, each equipped with its own set of tools to handle domain-specific tasks.

---

## Architecture

```
User Query
    |
    v
+----------------+
|  Router Agent  |  classifies query into one of 5 domains
+-------+--------+
        |
   +----+------------------------------------------+
   |         |           |           |              |
   v         v           v           v              v
Business/ Data       Member/     Order/         Product
Marketing Science    Fraud       Shipping    Recommendation
 Agent    Agent      Agent        Agent          Agent
```

The graph is built on LangGraph's `StateGraph`. The router drives conditional edges to the correct specialist node, and shared state is typed via `TypedDict` and passed through each step.

---

## Agents

### Business / Marketing

| Story | Task |
|-------|------|
| US1 | Black Friday Campaign ROI: identify top-performing ads and recommend budget reallocation |
| US2 | Q2 Content Strategy: surface high-engagement themes and propose a social calendar |
| US3 | Competitor Analysis: compare Peloton membership growth against NordicTrack and Echelon |

Tools: `get_campaign_analytics`, `get_content_engagement`, `get_competitor_data`

---

### Data Science

| Story | Task |
|-------|------|
| US1 | Cycling Performance: review last 30 classes and suggest output and cadence improvements |
| US2 | Workout Consistency: show 3-month patterns and recommend a personalized schedule |
| US3 | Demographic Report: popular class types by age group and instructor completion rates |

Tools: `get_cycling_performance`, `get_workout_consistency`, `get_demographic_report`

---

### Membership / Fraud Detection

| Story | Task |
|-------|------|
| US1 | Suspicious Device Activity: list recent sign-ins and revoke unrecognized devices |
| US2 | Password Reset: walk locked-out members through OTP-based account recovery |
| US3 | Membership Tier Comparison: compare All-Access vs App+ and recommend the right plan |

Tools: `list_recent_devices`, `revoke_device`, `initiate_password_reset`, `get_membership_tiers`

---

### Order / Shipping

| Story | Task |
|-------|------|
| US1 | Order Tracking: look up shipment status, carrier, and ETA |
| US2 | Return Process: initiate returns for damaged or defective items |
| US3 | Delayed Order Escalation: flag and escalate orders that have missed their ship date |

Tools: `track_order`, `initiate_return`, `escalate_delayed_order`

---

### Product Recommendation

| Story | Task |
|-------|------|
| US1 | Yoga Accessories: recommend mats and blocks based on recent class history |
| US2 | Bike vs Bike+: feature-by-feature comparison with a budget-based recommendation |
| US3 | Tread Cross-Sell: suggest accessory bundles for customers who recently bought a Tread |

Tools: `get_member_class_history`, `get_product_catalog`, `compare_products`, `get_cross_sell_bundles`

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| LLM | GPT-4o via `langchain-openai` |
| Agent Framework | LangChain 1.0 |
| Graph Orchestration | LangGraph `StateGraph` |
| Training Data | Custom JSON few-shot pairs with train/test split per agent |
| Notebook | Jupyter (Python 3) |

---

## Repo Structure

```
├── notebook/
│   └── peloton_ai_agents.ipynb    # full implementation, all 15 user stories
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

## Setup

1. **Install dependencies**
   ```bash
   pip install langchain langchain-openai langgraph openai python-dotenv
   ```

2. **Add your OpenAI API key**

   Either export it directly:
   ```bash
   export OPENAI_API_KEY="sk-..."
   ```
   Or add it to a `.env` file in the project root:
   ```
   OPENAI_API_KEY=sk-...
   ```

3. **Open the notebook**
   ```bash
   jupyter notebook notebook/peloton_ai_agents.ipynb
   ```

4. **Run all cells.** Section 8 runs all 15 user stories end-to-end. Section 9 runs router regression tests against the held-out testing data.

---

## Training Data

Each agent loads up to 4 domain-specific few-shot examples from `training_data/` via `load_training_pairs()`, which are injected into the system prompt before the agent processes a live query. The separate testing files are used only for router regression tests to prevent data leakage.

---

## Notes

- All tool responses use mock data that simulates realistic Peloton backend payloads.
- The `human_review` flag in `AgentState` is set automatically for sensitive operations like device revocation and order escalations.
- The graph can be rendered as a Mermaid diagram via `app.get_graph().draw_mermaid_png()`.
