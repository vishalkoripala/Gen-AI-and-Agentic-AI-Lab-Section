## 🚀 Live Application

👉 **[OPEN CASEFILE — LIVE APPLICATION](https://vishalkoripala.github.io/Gen-AI-and-Agentic-AI-Lab-Section/)**

> Try the live rule-based AI symptom reasoning agent directly in your browser.# CASEFILE --- AI Symptom Reasoning Agent

> **An educational rule-based AI agent demonstrating knowledge
> representation, forward chaining, weighted rule reasoning, and
> explainable decision making.**

## 📌 Project Overview

**CASEFILE** is a medical symptom-checking demonstration developed for
**Experiment 5 (CO2): Develop an AI agent with reasoning capability
using knowledge representation and rule-based decision making.**

The system accepts symptoms reported by a user and reasons over a
predefined weighted knowledge base. Instead of using a machine-learning
model for the core decision, CASEFILE uses explicit rules and an
inference engine to produce ranked possible conditions together with the
evidence that caused each rule to fire.

### Main components

1.  **Frontend --- `index.html`**
    -   Interactive symptom-selection interface
    -   Quick case presets
    -   Searchable symptom list
    -   Ranked reasoning results
    -   Confidence visualization
    -   Matched and missing evidence
    -   Explanation of why a rule fired
    -   Knowledge-base display
2.  **Reasoning Backend --- n8n workflow**
    -   Webhook receives symptoms from the frontend
    -   Knowledge base represents conditions and weighted symptoms
    -   Rule engine performs forward-chaining style inference
    -   Confidence is calculated for applicable rules
    -   Conclusions are ranked and returned as JSON

------------------------------------------------------------------------

## 🎯 Objective

The objective is to demonstrate how an AI agent can:

-   Represent domain knowledge explicitly.
-   Store facts supplied by a user.
-   Apply rule-based decision making.
-   Perform inference over those rules.
-   Calculate weighted confidence.
-   Explain why a conclusion was reached.
-   Return multiple ranked conclusions rather than a single opaque
    prediction.

------------------------------------------------------------------------

## 🧠 Knowledge Representation

CASEFILE uses a **weighted knowledge base**. Each condition is
represented by symptoms with numerical weights.

Example:

``` text
Common Cold
├── runny_nose       → 0.90
├── sneezing         → 0.85
├── sore_throat      → 0.60
├── mild_cough       → 0.60
├── nasal_congestion → 0.80
└── mild_headache    → 0.30
```

The user's selected symptoms become the system's working facts.

A rule is eligible to fire when:

``` text
Matched symptoms >= 2
AND
Confidence >= 0.30
```

The confidence calculation is:

``` text
confidence =
sum(weight of matched symptoms)
--------------------------------
sum(weight of all symptoms in rule)
```

The resulting conclusions are sorted from highest to lowest confidence.

------------------------------------------------------------------------

## 📚 Knowledge Base

The current knowledge base contains **10 conditions**:

  \#   Condition
  ---- -----------------
  1    Common Cold
  2    Influenza (Flu)
  3    COVID-19
  4    Malaria
  5    Typhoid Fever
  6    Dengue Fever
  7    Migraine
  8    Gastroenteritis
  9    Asthma
  10   Chickenpox

------------------------------------------------------------------------

## ⚙️ System Architecture

``` text
                    USER
                     │
                     ▼
              ┌──────────────┐
              │ CASEFILE UI  │
              │  index.html  │
              └──────┬───────┘
                     │
              HTTP POST symptoms
                     │
                     ▼
              ┌──────────────┐
              │ n8n Webhook  │
              └──────┬───────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Knowledge Base       │
          │ Weighted Rules       │
          └──────────┬───────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Rule / Inference      │
          │ Engine                │
          └──────────┬───────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Confidence + Ranking  │
          └──────────┬───────────┘
                     │
                     ▼
          ┌──────────────────────┐
          │ Reasoning Trace       │
          └──────────┬───────────┘
                     │
                     ▼
               JSON Response
                     │
                     ▼
              CASEFILE UI
```

------------------------------------------------------------------------

## 🔄 Agent Workflow

### 1. Perceive

The user selects symptoms through the CASEFILE interface.

### 2. Represent

The selected symptoms are converted into facts and sent to the n8n
workflow.

``` json
{
  "symptoms": [
    "fever",
    "dry_cough",
    "fatigue",
    "sore_throat"
  ]
}
```

### 3. Reason

The rule engine compares the facts against the weighted knowledge base.

### 4. Infer

Applicable rules are fired when they satisfy the minimum symptom and
confidence thresholds.

### 5. Rank

The resulting conditions are sorted according to confidence.

### 6. Explain

The system reports matched symptoms, missing typical symptoms, matched
weights, total rule weight, calculated confidence, and a suggested next
step.

------------------------------------------------------------------------

## 🧪 Example

### Input

``` text
runny nose
sneezing
sore throat
nasal congestion
```

### Reasoning

The system compares these facts against all ten conditions. The Common
Cold rule contains matching evidence for the reported symptoms.

### Output concept

``` text
CASE 01
Common Cold

Likelihood: High / Moderate / Low

Matched evidence:
✓ runny nose
✓ sneezing
✓ sore throat
✓ nasal congestion

Why this rule fired:
confidence = matched weight / total rule weight
```

------------------------------------------------------------------------

## 🧩 n8n Workflow

The supplied workflow contains three main stages:

``` text
Casefile Webhook
        ↓
Knowledge Base + Rule Engine
        ↓
Return Reasoning Result
```

### Casefile Webhook

Receives symptoms from `index.html`.

### Knowledge Base + Rule Engine

Contains the conditions, symptom weights, thresholds, matching logic,
confidence calculation, ranking, and reasoning information.

### Return Reasoning Result

Returns the final reasoning result to the frontend as JSON.

------------------------------------------------------------------------

## 📁 Project Structure

Recommended structure inside your existing projects repository:

``` text
CASEFILE/
│
├── index.html
│
├── n8n/
│   └── CASEFILE_n8n_workflow.json
│
└── README.md
```

------------------------------------------------------------------------

## 🚀 Setup

### Frontend

Use the supplied `CASEFILE_n8n.html` file as:

``` text
index.html
```

The frontend contains a placeholder for the n8n production webhook:

``` javascript
const N8N_WEBHOOK_URL =
  'PASTE_YOUR_N8N_PRODUCTION_WEBHOOK_URL_HERE';
```

Replace the placeholder with the production webhook URL generated by
your n8n instance.

### n8n

1.  Open your n8n instance.
2.  Import `n8n/CASEFILE_n8n_workflow.json`.
3.  Open the **Casefile Webhook** node.
4.  Confirm the HTTP method is `POST`.
5.  Use the path:

``` text
casefile-symptom-checker
```

6.  Activate the workflow.
7.  Copy the production webhook URL.
8.  Add that URL to `index.html`.

------------------------------------------------------------------------

## 🌐 Deployment

The frontend is a static HTML application and can be hosted using GitHub
Pages, Netlify, Cloudflare Pages, or another static hosting service.

The n8n workflow must run on an accessible n8n instance.

``` text
Static Hosting
      │
      │ index.html
      ▼
   CASEFILE
      │
      │ HTTPS POST
      ▼
  n8n Webhook
      │
      ▼
Rule-Based Engine
      │
      ▼
   JSON Result
      │
      ▼
   CASEFILE
```

------------------------------------------------------------------------

## 🔬 Experiment 5 Mapping

  Experiment Requirement       CASEFILE Implementation
  ---------------------------- -----------------------------------------
  AI Agent                     CASEFILE symptom reasoning agent
  Knowledge Representation     Weighted disease-symptom knowledge base
  Facts                        User-selected symptoms
  Rule-Based Decision Making   Explicit condition/symptom rules
  Reasoning                    Forward-chaining style rule evaluation
  Inference Engine             n8n Code node
  Confidence                   Weighted matched-symptom score
  Explainability               Matched evidence + calculation
  Decision Output              Ranked possible conditions

------------------------------------------------------------------------

## 🛡️ Safety and Scope

**CASEFILE is an educational demonstration and is not a diagnostic
device.**

The system uses a limited predefined knowledge base and cannot replace
professional medical evaluation, laboratory testing, or clinical
judgment.

The conditions and outputs should therefore be interpreted as
**rule-based educational possibilities**, not medical diagnoses.

For real symptoms, especially severe or rapidly worsening symptoms,
professional medical care should be sought.

------------------------------------------------------------------------

## 💡 Why This Demonstrates Reasoning

CASEFILE does not merely map one symptom to one condition. It:

1.  Receives multiple facts.
2.  Compares those facts against multiple rules.
3.  Calculates weighted evidence.
4.  Applies thresholds.
5.  Fires applicable rules.
6.  Ranks conclusions.
7.  Shows the evidence behind the conclusions.

This makes the reasoning process observable and suitable for
demonstrating **knowledge representation and rule-based AI reasoning**.

------------------------------------------------------------------------

## 🛠️ Technologies

-   HTML5
-   CSS3
-   JavaScript
-   n8n
-   Webhooks
-   Rule-based inference
-   Weighted knowledge representation
-   Forward-chaining style reasoning

------------------------------------------------------------------------

## 📌 Current Limitations

-   The knowledge base is predefined and limited to 10 conditions.
-   The system is not a clinical diagnostic system.
-   Reasoning quality depends on manually defined rules and weights.
-   It does not learn new rules automatically.
-   The n8n workflow must be accessible to the deployed frontend.
-   A production deployment should configure appropriate access control
    and security for the workflow endpoint.

------------------------------------------------------------------------

## 🔮 Future Improvements

-   Expand the knowledge base.
-   Add more detailed rule relationships.
-   Add certainty factors and rule priorities.
-   Add multi-step inference.
-   Add persistent case history.
-   Add an administrative rule-management interface.
-   Add a visual inference graph.
-   Add audit logs for every reasoning step.
-   Add authentication and rate limiting to the backend.
-   Add a clinician-reviewed knowledge base for any real-world
    application.

------------------------------------------------------------------------

## 📄 Academic Context

**Experiment:** 5\
**CO:** CO2\
**Topic:** Develop an AI agent with reasoning capability using knowledge
representation and rule-based decision making.\
**Case Study:** Medical symptom checker using rule-based logic to
suggest possible illnesses based on patient symptoms.

------------------------------------------------------------------------

## ⚠️ Disclaimer

This project is created for academic and educational purposes. It does
not provide medical diagnosis or medical advice.
