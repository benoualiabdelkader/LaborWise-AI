# ⚖️ LaborWise AI

**An evidence-grounded HR & Labor Compliance AI agent for Algerian labor law, built with [NeuralSeek mAIstro](https://neuralseek.com).**

Built for the **#mAIstro2026 Intern Challenge** · @NeuralSeek · #intern

> In HR & Labor Compliance, guessing is a liability. LaborWise AI answers only from retrieved legal evidence, resolves conflicts between sources by a fixed authority order, and refuses to answer when the evidence is not there.

▶️ **Demo video:** `https://www.linkedin.com/in/benouali-abdelkader-yahia-zakaria-4a917a247/`

---

## Table of contents

1. [The problem](#1-the-problem)
2. [Solution overview](#2-solution-overview)
3. [5-tier source authority logic](#3-5-tier-source-authority-logic)
4. [Architecture: the mAIstro pipeline](#4-architecture-the-maistro-pipeline)
5. [Step-by-step build guide](#5-step-by-step-build-guide)
6. [Prompts](#6-prompts)
7. [Live test results](#7-live-test-results)
8. [Guardrails and limitations](#8-guardrails-and-limitations)
9. [Repository structure](#9-repository-structure)
10. [Lessons learned](#10-lessons-learned)

---

## 1. The problem

General-purpose LLMs fail in labor compliance in three predictable ways:

| Failure | Example |
|---|---|
| **Hallucinated numbers** | Inventing an overtime percentage or leave entitlement |
| **Confused policies** | Mixing rules from different countries or documents |
| **Outdated labor law** | Quoting a repealed or amended provision |

An HR assistant that is "usually right" is not usable when a wrong figure can become a legal dispute.

## 2. Solution overview

LaborWise AI is a multi-tier, retrieval-grounded agent:

- **Grounded:** every answer is built from records retrieved from a Knowledge Base of official Algerian legislation and Ministry of Labor guidance.
- **Deterministic:** when sources disagree, a fixed 5-tier authority order decides which one wins.
- **Refuses to guess:** if the retrieved evidence does not support an answer, the agent says so.
- **Traceable:** answers cite the legal text and tier they came from.

## 3. 5-tier source authority logic

Every source in the Knowledge Base is assigned an authority tier. When two sources conflict, the **lower-numbered tier wins**.

| Tier | Source type | Role |
|---|---|---|
| **Tier 1** | Primary national law (e.g. Law No. 90-11 of 21 April 1990 on labor relations; Ordinance No. 97-03 on the legal duration of work) | Binding legal text. Always prevails |
| **Tier 2** | Institutional guidance (Ministry of Labor, Employment and Social Security, MTESS) | Official interpretation and guidelines |
| **Tier 3** | `[TODO: describe, e.g. regulations / executive decrees]` | `[TODO]` |
| **Tier 4** | `[TODO: describe, e.g. collective agreements]` | `[TODO]` |
| **Tier 5** | `[TODO: describe, e.g. company policy / secondary sources]` | `[TODO]` |

> Fill in Tiers 3–5 to match the documents you actually loaded.

**Resolution rules**

1. Prefer the highest-authority tier that addresses the question.
2. Never blend a lower-tier statement into a number that a higher tier already defines.
3. If tiers conflict, state the higher-tier rule and note the conflict.
4. If no tier supports an answer, refuse and say what is missing.

## 4. Architecture: the mAIstro pipeline

```
User question
     │
     ▼
┌───────────┐   ┌──────────────┐   ┌───────────────────────┐   ┌─────────┐   ┌─────────────┐
│ KB Search │ → │ Set Variable │ → │ LLM Plan (Llama 3.1   │ → │ LLM Act │ → │ Send to LLM │
│           │   │ (kbResults)  │   │ 8B): JSON query plan  │   │ (rules) │   │ (synthesis) │
└───────────┘   └──────────────┘   └───────────────────────┘   └─────────┘   └─────────────┘
                                                                                    │
                                                                                    ▼
                                                                        Evidence-grounded answer
```

| Node | Purpose |
|---|---|
| **KB Search** | Retrieves relevant records from the Knowledge Base for the user's `task` |
| **Set Variable** | Stores the results as `kbResults` for use in later prompts |
| **LLM Plan** (Llama 3.1 8B) | Creates a structured JSON execution plan for the query |
| **LLM Act** | Applies the guardrails and source-authority rules to the retrieved records |
| **Send to LLM** | Synthesizes the final answer strictly from the verified analysis |

## 5. Step-by-step build guide

### Step 1: Prepare the knowledge sources
1. Collect the official Algerian labor texts (labor relations law, working-time ordinance, Ministry of Labor guidance).
2. Clean and tag each document with its **tier** (1–5) and **source name**.
3. Keep tier and source in the document text or metadata so the agent can cite them.

### Step 2: Create the Knowledge Base in NeuralSeek
1. Open NeuralSeek and go to **KnowledgeBase** setup.
2. Connect or upload your documents.
3. Confirm the documents are indexed and searchable.

### Step 3: Create the agent in mAIstro
1. Open **mAIstro → Agent Builder** and create a new agent (`LabourWise_AI`).
2. Set the agent parameter **`task`** (the user's question).

### Step 4: Add the pipeline nodes
Add them in this order and connect them:
1. **NS KB Search**: query `<< name: task, prompt: true >>`; set the number of results and any filters.
2. **Set a Variable**: save the search results as `kbResults`.
3. **LLM Plan**: model `llama-3.1-8b-instruct`; produce a structured JSON plan (see [Prompts](#6-prompts)).
4. **LLM Act**: enforce the guardrails and tier rules against `<< name: kbResults, prompt: false >>`.
5. **Send to LLM**: write the final answer using only the verified analysis.

### Step 5: Add guardrails
Put the rules from [Guardrails](#8-guardrails-and-limitations) into the LLM Act and Send to LLM prompts. Turn **Stream** and **Auto-Enhance** on or off as needed for your tests.

### Step 6: Test
Run the agent from the **Evaluate** button with the test questions in [Live test results](#7-live-test-results). Check that each figure appears in the retrieved records.

## 6. Prompts

> Replace each block with the exact prompt text from your agent.

**System identity (Send to LLM)**
```
You are LabourWise_AI, an evidence-grounded Multi-Jurisdiction Labour Rights & HR Policy Assistant.
Answer STRICTLY AND EXCLUSIVELY from the Retrieved Knowledge Base Records.
```

**LLM Plan**
```
[TODO: paste your Plan prompt. It should output a structured JSON execution plan.]
```

**LLM Act**
```
[TODO: paste your Act prompt: tier rules, conflict resolution, refusal rules.]
```

**Send to LLM**
```
[TODO: paste your final synthesis prompt.]
```

## 7. Live test results

| # | Question | Result |
|---|---|---|
| 1 | How is paid annual leave calculated in Algeria, and what additional leave applies to workers in the southern regions? | `[TODO: paste the answer]` |
| 2 | What is the maximum allowed overtime percentage in Algeria, the minimum overtime pay increase, and the legal hours and restrictions for night work? | Maximum overtime: **20%** of the legal working duration. Minimum overtime pay increase: **50%** of the regular hourly wage. Legal hours: **40 hours/week** over 5 working days, max **12 hours/day**. Night work: **21:00–05:00**; prohibited for workers under 19. Sources cited: Law No. 90-11 (Tier 1), Ordinance No. 97-03 (Tier 1) |

Add screenshots to `/docs/images/` and link them here.

## 8. Guardrails and limitations

**Guardrails**
- Answer only from retrieved records; never from model memory.
- Quote figures exactly as they appear in the evidence.
- Apply the tier order when sources conflict.
- Refuse when evidence is missing, and say what is missing.
- Do not give personal legal advice; add a short disclaimer.

**Limitations**
- Accuracy depends on the Knowledge Base. Laws change, so documents need periodic review.
- Coverage is limited to the documents loaded.
- This is an informational tool, not a substitute for a qualified lawyer.

## 9. Repository structure

```
LaborWise-AI/
├── README.md
├── prompts/
│   ├── plan.txt
│   ├── act.txt
│   └── synthesis.txt
├── kb-sources/          # list of documents and their tiers (not the files if restricted)
└── docs/images/         # screenshots of the pipeline and tests
```

## 10. Lessons learned

- The structure around the model (retrieval, planning, rules) matters as much as the model.
- A small model like Llama 3.1 8B works well when it has a clear plan and strict rules.
- A confident wrong answer is worse than a refusal in compliance.

---

**Built by** `[your name]` · [LinkedIn](`[link]`) · Powered by NeuralSeek mAIstro · #mAIstro2026 #intern
