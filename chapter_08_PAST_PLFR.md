# Chapter 8: PAST, PLFR, and Structured Prompt Frameworks

## Learning Outcomes

After reading this chapter, a student will understand structured prompt frameworks (PAST and PLFR) well enough to design task decomposition architectures for agentic pipelines without making the mistake of relying on unstructured natural language prompts that silently degrade under task complexity.

Specifically, students will be able to:

1. **Analyze** why unstructured prompts fail under compositional task load and identify the exact point of degradation (Bloom's: Analyze).
2. **Apply** the PAST (Problem, Action, Steps, Task) and PLFR (Prompt, Logic, Format, Result) frameworks to decompose professional tasks into model-interpretable instruction sequences (Bloom's: Apply).
3. **Evaluate** when PAST vs. PLFR is the appropriate framework given task characteristics — and what you sacrifice with each choice (Bloom's: Evaluate).
4. **Design** agentic pipelines where PAST serves as the task decomposition backbone and PLFR governs agent-to-agent communication contracts (Bloom's: Create).
5. **Diagnose** the failure mode where a structurally valid prompt produces confidently wrong outputs because the framework enforced format without enforcing reasoning (Bloom's: Evaluate).

---

## 8.1 The Scenario: When "Just Ask the Model" Breaks

Consider a data engineering team at a mid-size fintech company. Their analysts send ad-hoc requests to an internal LLM-powered assistant: "Pull the Q3 churn numbers, compare them to Q2, and write a summary for the VP of Product." This prompt works — once. The model returns a coherent paragraph with reasonable numbers.

Then the requests get harder. "Pull Q3 churn segmented by plan tier, compare to Q2 and Q1, flag any segment where churn exceeds 5%, cross-reference with NPS scores from the CX team's last survey, and draft a slide deck summary with recommendations." The model still returns something. It looks confident. But the segment comparison is against the wrong quarter. The NPS cross-reference is fabricated. The recommendations contradict the data.

Nothing in the model changed. The architecture of the prompt changed — or rather, it didn't change when the task complexity did. This is the failure mode this chapter addresses: **prompt ambiguity scales with task complexity, and unstructured natural language prompts provide no mechanism to detect or contain that scaling.**

The model is not the problem. The architecture of the instruction is.

---

## 8.2 The Mechanism: Structured Decomposition as Architectural Scaffolding

### 8.2.1 Why Structure Matters: The Ambiguity Multiplier

An unstructured prompt is a single string of natural language that the model must simultaneously parse for intent, scope, sequence, constraints, and output format. Each additional sub-task in the prompt introduces ambiguity at every one of those dimensions. The ambiguity doesn't add — it multiplies.

A prompt with three sub-tasks doesn't have 3x the ambiguity of a single-task prompt. It has closer to 3^n ambiguity, where n is the number of unstated dependencies between sub-tasks. "Compare Q3 to Q2" presupposes the model knows which metric, which segmentation, and which data source. Stack three such presuppositions and the model is choosing from a combinatorial space of interpretations — with no mechanism to surface the choice it made.

![Figure 1 — The Ambiguity Multiplier](figures/fig1_ambiguity_multiplier.svg)

*Figure 1: As sub-tasks increase, the interpretation space grows exponentially (≈3ⁿ). A single-task prompt has roughly 3 plausible interpretations; a three-task prompt with unstated dependencies has 27. Structured frameworks exist to collapse this space before the model encounters it.*

Structured prompt frameworks solve this by making the decomposition explicit. They impose a **logical scaffold** that separates intent from sequence from format, forcing the prompt author to resolve ambiguities before the model encounters them.

### 8.2.2 PAST: Problem → Action → Steps → Task

PAST decomposes a prompt into four explicit layers:

| Component | Function | Architectural Role |
|-----------|----------|--------------------|
| **Problem** | States the context and the specific challenge | Scopes the model's reasoning space |
| **Action** | Defines what the model must do (verb-level) | Constrains the operation type |
| **Steps** | Enumerates the sequential sub-operations | Linearizes the execution plan |
| **Task** | Specifies the concrete deliverable and format | Anchors the output contract |

The architectural insight is that PAST **linearizes task execution**. By forcing the prompt author to enumerate steps, PAST converts a parallel-ambiguous instruction ("do all of this") into a sequential-explicit one ("do this, then this, then this"). The model processes a queue, not a cloud.

![Figure 2 — PAST Framework Anatomy](figures/fig2_past_anatomy.svg)

*Figure 2: The PAST framework as a vertical pipeline. Problem scopes the reasoning space, Action constrains the operation, Steps linearizes execution into a numbered queue, and Task anchors the output contract. The Steps layer is expanded to show sequential sub-boxes — this is the layer that converts ambiguity into sequence. Strength: completeness. Weakness: no reasoning constraint.*

**Example — Unstructured:**
> "Analyze our Q3 churn data segmented by plan tier, compare to prior quarters, and draft recommendations."

**Example — PAST-Structured:**
> **Problem:** Our Q3 churn rate increased across multiple plan tiers. Leadership needs a segmented analysis to identify which tiers are driving the increase and what actions to take.
>
> **Action:** Perform a comparative churn analysis and generate actionable recommendations.
>
> **Steps:**
> 1. Calculate Q3 churn rate for each plan tier (Free, Pro, Enterprise).
> 2. Retrieve Q2 and Q1 churn rates for the same tiers.
> 3. Compute quarter-over-quarter delta for each tier.
> 4. Flag any tier where Q3 churn exceeds 5%.
> 5. For flagged tiers, identify the top contributing factor from available data.
> 6. Draft one recommendation per flagged tier.
>
> **Task:** Produce a table with columns [Tier, Q1 Churn, Q2 Churn, Q3 Churn, QoQ Delta, Flag] followed by a bulleted list of recommendations, each tied to a specific tier.

The PAST version eliminates ambiguity about which quarters to compare, what threshold triggers a flag, and what the output format looks like. The model's job shifts from "interpret my intent" to "execute my plan."

### 8.2.3 PLFR: Prompt → Logic → Format → Result

PLFR operates on a different axis. Where PAST linearizes execution, PLFR **contracts the reasoning chain**:

| Component | Function | Architectural Role |
|-----------|----------|--------------------|
| **Prompt** | The core instruction or question | Defines the reasoning entry point |
| **Logic** | The reasoning method or analytical framework to apply | Constrains *how* the model thinks |
| **Format** | The output structure, schema, or template | Enforces output predictability |
| **Result** | The expected characteristics of a correct answer | Establishes a validation benchmark |

PLFR's architectural contribution is the **Logic** component. PAST tells the model *what* to do step-by-step. PLFR tells the model *how to reason*. This distinction becomes critical in agentic systems where agents must not only execute tasks but justify their outputs to downstream agents.

![Figure 3 — PLFR Framework Anatomy](figures/fig3_plfr_anatomy.svg)

*Figure 3: The PLFR framework. The Logic layer is visually dominant because it is architecturally dominant — it is the component that distinguishes PLFR from a simple format template. The feedback arrow from Result back to Logic enables self-validation against the reasoning method. Compare with Figure 2: PAST constrains WHAT the model does; PLFR constrains HOW it reasons.*

**Example — PLFR-Structured:**
> **Prompt:** Determine which plan tier is most responsible for our Q3 churn increase.
>
> **Logic:** Use a contribution-weighted analysis. For each tier, multiply the tier's churn rate delta (Q3 minus Q2) by the tier's share of total subscribers. The tier with the highest weighted contribution is the primary driver.
>
> **Format:** Return a JSON object: `{"primary_driver": "<tier>", "weighted_contribution": <float>, "breakdown": [{"tier": "<n>", "delta": <float>, "weight": <float>, "contribution": <float>}]}`
>
> **Result:** The primary_driver field should identify exactly one tier. The weighted_contribution values should sum to approximately the total churn delta. If two tiers are within 0.5pp of each other, flag the result as "ambiguous" and return both.

Notice what PLFR does that PAST does not: it specifies the *analytical method*. The model isn't free to choose between absolute comparison, percentage comparison, or weighted contribution — the Logic component locks it to one method. This makes the output reproducible and auditable.

---

## 8.3 The Design Decision: When to Use PAST vs. PLFR

This is not a "pick your favorite" situation. PAST and PLFR solve different problems and fail in different ways.

### Decision Criteria

| Criterion | Use PAST | Use PLFR |
|-----------|----------|----------|
| Task type | Multi-step procedural execution | Analytical reasoning with method constraints |
| Primary risk | Step omission or reordering | Incorrect reasoning method |
| Output variability | High variance in completeness | High variance in analytical approach |
| Agentic role | Task decomposition for planning agents | Communication contract between reasoning agents |
| Best for | Report generation, data pipelines, code review workflows | Classification, diagnosis, scoring, evaluation |

![Figure 4 — PAST vs. PLFR Decision Landscape](figures/fig4_decision_landscape.svg)

*Figure 4: PAST (left) operates as structural scaffolding — it constrains what the model does. PLFR (right) operates as a reasoning network — it constrains how the model thinks. For complex agentic tasks (center of spectrum), both frameworks are required in a layered architecture.*

### The Complementary Architecture

In production agentic systems, PAST and PLFR are not alternatives — they are layers. The planning agent uses PAST to decompose a user request into sub-tasks. Each sub-task is dispatched to an execution agent using a PLFR-formatted instruction that specifies the reasoning method and output contract.

![Figure 5 — Two-Layer Agentic Architecture](figures/fig5_two_layer_architecture.svg)

*Figure 5: The core architectural claim. The Planning Agent (top, steel blue) uses PAST to decompose the user request into sequential steps. Each step is dispatched to an Execution Agent (bottom, amber) wrapped in a PLFR contract that specifies the reasoning method, output format, and validation criteria. This two-layer nesting — PAST for decomposition, PLFR for execution — is what prevents the "structured but wrong" failure.*

This two-layer architecture — PAST for decomposition, PLFR for execution — is the chapter's core architectural claim. Neither framework alone is sufficient. PAST without PLFR produces correctly sequenced but methodologically unconstrained outputs. PLFR without PAST produces methodologically sound but poorly scoped responses to complex tasks.

---

## 8.4 The Failure Case: Structure Without Reasoning

Here is the failure mode that this architecture is designed to prevent — and the one that PAST alone **cannot** prevent.

### The "Confidently Structured, Silently Wrong" Failure

A PAST-structured prompt enforces format and sequence. It does not enforce reasoning. When the Steps component contains an instruction like "Compare Q3 to Q2," PAST ensures the model will perform a comparison. It does not ensure the model will choose the right comparison method.

**Triggering the failure:**

```
PAST Prompt:
  Problem: Customer churn increased in Q3.
  Action: Identify the primary driver.
  Steps:
    1. List churn rates by tier.
    2. Compare Q3 to Q2.
    3. Identify the tier with the highest churn.
  Task: Return the tier name and churn rate.
```

This prompt has a structural defect: Step 3 says "highest churn" when the business question is "highest *increase* in churn." A tier with 12% churn in both Q2 and Q3 (no change) will outrank a tier that went from 2% to 8% (massive increase). The PAST framework faithfully executes the steps as written — and returns the wrong answer with perfect formatting.

The model will not flag this. The output will be clean, tabular, and wrong. The PAST framework gave the model a sequence to follow, and it followed it. The reasoning error is in the steps, not in the model.

**The fix requires PLFR's Logic component:**

```
PLFR addition to Step 2:
  Logic: Compare using absolute delta (Q3 rate minus Q2 rate),
         not absolute level. The driver is the tier with the
         largest positive delta, not the highest rate.
```

This is the architectural argument: **structure without reasoning specification is a false guarantee.** PAST provides the skeleton. PLFR provides the brain. An agentic pipeline that uses PAST alone for task decomposition will produce outputs that *look* correct because they follow the prescribed format, while silently applying the wrong analytical method at every reasoning step.

![Figure 6 — The "Confidently Structured, Silently Wrong" Failure](figures/fig6_failure_mode.svg)

*Figure 6: The same churn data flows through two pipelines. The PAST-only pipeline (left, amber) follows "identify highest churn" and returns Enterprise at 12% — the highest absolute rate. The PAST+PLFR pipeline (right, blue) applies the Logic component "use absolute delta" and returns Free at +5.5pp — the highest churn increase. The business decisions are opposite. Red ✗ and green ✓ indicators use shapes (not just color) for accessibility.*

### Observing the Failure in Practice

In the accompanying notebook, we demonstrate this failure with a concrete dataset. The PAST-only prompt returns "Enterprise" as the primary churn driver (highest absolute rate). The PAST+PLFR prompt returns "Free" (highest rate *increase*). The business decision these two answers imply are opposite: one says focus retention on Enterprise, the other says investigate what changed for Free-tier users.

The failure is not a hallucination. It is a correct execution of an under-specified instruction. That is what makes it dangerous — and what makes the architecture, not the model, the leverage point.

---

## 8.5 Scaling to Agentic Pipelines

### 8.5.1 PAST as the Planning Agent's Backbone

In a LangGraph or similar orchestration framework, the planning agent receives a user request and must produce an execution plan. PAST provides the structural template:

```python
PLANNING_PROMPT = """
You are a task planning agent. Decompose the user request using PAST:

Problem: {Restate the user's problem in one sentence.}
Action: {Define the primary operation in verb form.}
Steps: {Enumerate 3-7 sequential sub-tasks. Each step must be
        independently executable by a downstream agent.}
Task: {Specify the final deliverable format.}

Rules:
- Each Step must have exactly one verb and one object.
- No step may depend on a step more than 2 positions ahead.
- If a step requires data from a prior step, state the dependency.
"""
```

The constraint "each step must be independently executable" is the design decision. It means the planning agent produces a DAG of sub-tasks, not a prose description. Each node in the DAG is dispatchable.

### 8.5.2 PLFR as the Agent-to-Agent Communication Contract

When the planning agent dispatches a sub-task to an execution agent, it wraps the instruction in a PLFR envelope:

```python
EXECUTION_PROMPT = """
Prompt: {sub_task_description}
Logic: {reasoning_method — e.g., "weighted contribution analysis",
        "threshold-based classification", "comparative delta"}
Format: {output_schema — JSON with specified fields}
Result: {validation_criteria — what a correct output looks like}
"""
```

The PLFR envelope serves as an **interface contract** between agents. The downstream agent knows:
- What to compute (Prompt)
- How to compute it (Logic)
- What shape the output must take (Format)
- How to self-validate (Result)

Without this contract, agent-to-agent communication degrades to unstructured natural language — and the ambiguity multiplier from Section 8.2.1 reappears, now compounded across agents.

### 8.5.3 The Human Decision Node

In the demo notebook, the AI scaffold proposes an agent coordination structure based on the PAST decomposition. Before the system finalizes the pipeline, it halts at a **Mandatory Human Decision Node**:

```python
# ══════════════════════════════════════════════════════════
# MANDATORY HUMAN DECISION NODE
# ══════════════════════════════════════════════════════════
# The AI proposed the following PAST decomposition:
#   Step 1: Extract churn data (data_agent)
#   Step 2: Compute deltas (analysis_agent)
#   Step 3: Cross-reference NPS (analysis_agent)
#   Step 4: Generate recommendations (writing_agent)
#
# ARCHITECTURAL ASSUMPTION: Steps 2 and 3 can execute in
# parallel because they read from Step 1 output only.
#
# VERIFY: Does Step 3 (NPS cross-reference) actually need
# the delta values from Step 2? If yes, these steps are
# sequential, not parallel, and the pipeline topology changes.
#
# YOUR DECISION: [parallel / sequential]
# YOUR REASONING: ___________________________________________
# ══════════════════════════════════════════════════════════
```

![Figure 7 — Pipeline Topology: Human Decision Node](figures/fig7_human_decision_node.svg)

*Figure 7: The AI proposed parallel execution (left, dashed amber border) for Steps 2 and 3. The Human Decision Node (center diamond) detects a hidden semantic dependency: Step 3 references "flagged tiers," computed in Step 2. The human overrides to sequential execution (right, solid blue border). The green arrow highlights the corrected dependency. The AI parsed dependencies syntactically; the human traced them semantically.*

This is the architectural moment where the human overrides or confirms the AI's structural proposal. In the video, this is where the author says: "The AI proposed parallel execution. I rejected it because the NPS cross-reference needs to know *which* tiers are flagged, which requires the delta computation to finish first. The correct topology is sequential."

---

## 8.6 Exercise: Breaking the System

**Your task:** Clone the notebook and modify Step 2 of the PAST decomposition to say "Compare Q3 to Q2" without specifying the comparison method. Run both the PAST-only and PAST+PLFR pipelines.

1. Observe that the PAST-only pipeline returns a result based on absolute churn rates.
2. Observe that the PAST+PLFR pipeline returns a result based on churn rate deltas.
3. Compare the business decisions implied by each result.
4. Now remove the Logic component from the PLFR execution prompt and re-run. Does the execution agent default to absolute comparison or delta comparison? Run it five times. Is the method consistent?

**What you should find:** Without the Logic component, the model's choice of comparison method is non-deterministic. Some runs use absolute rates, some use deltas, and some use percentage change. The output format remains consistent (because Format is still specified), but the analytical substance varies. This is the "confidently structured, silently wrong" failure in action.

**Open question:** Can the Result component of PLFR serve as a runtime validator — i.e., can the agent check its own output against the Result specification and flag inconsistencies? What are the limitations of self-validation when the reasoning method itself is the error?

---

## 8.7 Key Takeaways

1. **Unstructured prompts do not scale.** Ambiguity multiplies with task complexity, and natural language provides no containment mechanism.

2. **PAST linearizes execution.** It converts parallel-ambiguous instructions into sequential-explicit plans. Its strength is completeness; its weakness is that it constrains *what* the model does without constraining *how* it reasons.

3. **PLFR contracts reasoning.** It specifies the analytical method, output schema, and validation criteria. Its strength is reproducibility; its weakness is that it assumes the task is already scoped.

4. **The architectural pattern is layered: PAST for decomposition, PLFR for execution.** Neither framework alone is sufficient. Together, they provide both the execution plan and the reasoning contract.

5. **The failure mode is structure without reasoning specification.** A prompt can be perfectly formatted, correctly sequenced, and still produce wrong answers because the reasoning method was left to the model's discretion. Architecture — not the model — is the leverage point.

---

*Chapter 8 of Design of Agentic Systems with Case Studies. The master argument of this book is that architecture is the leverage point, not the model. This chapter demonstrates that claim through the specific case of prompt architecture: the structure you impose on instructions determines the quality of outputs more than the model you choose to execute them.*
