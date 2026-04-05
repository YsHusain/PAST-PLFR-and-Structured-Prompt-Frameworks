![fig7_human_decision_node](https://github.com/user-attachments/assets/67c4d31f-ec06-4eff-be8c-c58228f647b2)![fig6_failure_mode](https://github.com/user-attachments/assets/e7774a21-37c9-4d86-880e-e7129bb3a423)![fig5_two_layer_architecture](https://github.com/user-attachments/assets/24972c88-6a07-4f14-8ef4-50835a0b65b4)![fig4_decision_landscape](https://github.com/user-attachments/assets/196eb053-ae6c-465b-832b-a679cadc6452)![fig3_plfr_anatomy](https://github.com/user-attachments/assets/ff2f78ed-9f30-4e98-a8d0-2de0d51e0175)![fig1_ambiguity_multiplier](https://github.com/user-attachments/assets/bce422f2-a44d-4d8f-ba56-0b0fbc6e3895)# Chapter 8: PAST, PLFR, and Structured Prompt Frameworks

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

![<svg viewBox="0 0 800 450" xmlns="http://www.w3.org/2000/svg" font-family="Arial, Helvetica, sans-serif">
  <rect width="800" height="450" fill="white"/>

  <!-- Title -->
  <text x="400" y="30" text-anchor="middle" font-size="16" font-weight="bold" fill="#333">Figure 1 — The Ambiguity Multiplier</text>

  <!-- Single Task (Left) -->
  <text x="110" y="65" text-anchor="middle" font-size="12" font-weight="bold" fill="#4682B4">Single-Task Prompt</text>
  <rect x="60" y="75" width="100" height="40" rx="4" fill="#4682B4" opacity="0.9"/>
  <text x="110" y="100" text-anchor="middle" font-size="11" fill="white">Task 1</text>
  <!-- Single interpretation -->
  <line x1="110" y1="115" x2="110" y2="150" stroke="#D4A04A" stroke-width="2"/>
  <circle cx="110" cy="160" r="8" fill="#D4A04A" opacity="0.6"/>
  <text x="110" y="185" text-anchor="middle" font-size="10" fill="#6B7B8D">1 interpretation</text>

  <!-- Three-Task (Center) -->
  <text x="350" y="65" text-anchor="middle" font-size="12" font-weight="bold" fill="#4682B4">Three-Task Prompt</text>
  <rect x="290" y="75" width="50" height="30" rx="3" fill="#4682B4" opacity="0.9"/>
  <text x="315" y="95" text-anchor="middle" font-size="9" fill="white">T1</text>
  <rect x="345" y="75" width="50" height="30" rx="3" fill="#4682B4" opacity="0.8"/>
  <text x="370" y="95" text-anchor="middle" font-size="9" fill="white">T2</text>
  <rect x="400" y="75" width="50" height="30" rx="3" fill="#4682B4" opacity="0.7"/>
  <text x="425" y="95" text-anchor="middle" font-size="9" fill="white">T3</text>

  <!-- Branching lines from T1 -->
  <line x1="315" y1="105" x2="280" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>
  <line x1="315" y1="105" x2="315" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>
  <line x1="315" y1="105" x2="350" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>

  <!-- Branching lines from T2 -->
  <line x1="370" y1="105" x2="340" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>
  <line x1="370" y1="105" x2="370" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>
  <line x1="370" y1="105" x2="400" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>

  <!-- Branching from T3 -->
  <line x1="425" y1="105" x2="395" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>
  <line x1="425" y1="105" x2="425" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>
  <line x1="425" y1="105" x2="455" y2="135" stroke="#D4A04A" stroke-width="1.5" opacity="0.7"/>

  <!-- Sub-branches (level 2) - show explosion -->
  <g opacity="0.4" stroke="#D4A04A" stroke-width="1">
    <line x1="280" y1="135" x2="265" y2="160"/><line x1="280" y1="135" x2="280" y2="160"/><line x1="280" y1="135" x2="295" y2="160"/>
    <line x1="315" y1="135" x2="300" y2="160"/><line x1="315" y1="135" x2="315" y2="160"/><line x1="315" y1="135" x2="330" y2="160"/>
    <line x1="350" y1="135" x2="335" y2="160"/><line x1="350" y1="135" x2="350" y2="160"/><line x1="350" y1="135" x2="365" y2="160"/>
    <line x1="340" y1="135" x2="325" y2="160"/><line x1="340" y1="135" x2="340" y2="160"/><line x1="340" y1="135" x2="355" y2="160"/>
    <line x1="370" y1="135" x2="360" y2="160"/><line x1="370" y1="135" x2="370" y2="160"/><line x1="370" y1="135" x2="380" y2="160"/>
    <line x1="400" y1="135" x2="385" y2="160"/><line x1="400" y1="135" x2="400" y2="160"/><line x1="400" y1="135" x2="415" y2="160"/>
    <line x1="395" y1="135" x2="385" y2="160"/><line x1="395" y1="135" x2="395" y2="160"/><line x1="395" y1="135" x2="405" y2="160"/>
    <line x1="425" y1="135" x2="415" y2="160"/><line x1="425" y1="135" x2="425" y2="160"/><line x1="425" y1="135" x2="435" y2="160"/>
    <line x1="455" y1="135" x2="445" y2="160"/><line x1="455" y1="135" x2="455" y2="160"/><line x1="455" y1="135" x2="465" y2="160"/>
  </g>

  <text x="370" y="185" text-anchor="middle" font-size="10" fill="#6B7B8D">3³ = 27 interpretations</text>

  <!-- Inset Chart (Right) -->
  <rect x="550" y="60" width="220" height="170" rx="4" fill="none" stroke="#6B7B8D" stroke-width="1"/>
  <text x="660" y="80" text-anchor="middle" font-size="11" font-weight="bold" fill="#333">Interpretation Space Growth</text>

  <!-- Chart axes -->
  <line x1="590" y1="200" x2="750" y2="200" stroke="#6B7B8D" stroke-width="1"/>
  <line x1="590" y1="200" x2="590" y2="95" stroke="#6B7B8D" stroke-width="1"/>

  <!-- X-axis labels -->
  <text x="600" y="215" text-anchor="middle" font-size="9" fill="#6B7B8D">1</text>
  <text x="635" y="215" text-anchor="middle" font-size="9" fill="#6B7B8D">2</text>
  <text x="670" y="215" text-anchor="middle" font-size="9" fill="#6B7B8D">3</text>
  <text x="705" y="215" text-anchor="middle" font-size="9" fill="#6B7B8D">4</text>
  <text x="740" y="215" text-anchor="middle" font-size="9" fill="#6B7B8D">5</text>
  <text x="670" y="230" text-anchor="middle" font-size="10" fill="#6B7B8D">Sub-tasks</text>

  <!-- Y-axis labels -->
  <text x="580" y="200" text-anchor="end" font-size="9" fill="#6B7B8D">3</text>
  <text x="580" y="180" text-anchor="end" font-size="9" fill="#6B7B8D">9</text>
  <text x="580" y="160" text-anchor="end" font-size="9" fill="#6B7B8D">27</text>
  <text x="580" y="140" text-anchor="end" font-size="9" fill="#6B7B8D">81</text>
  <text x="580" y="105" text-anchor="end" font-size="9" fill="#6B7B8D">243</text>

  <!-- Exponential curve -->
  <polyline points="600,197 635,193 670,180 705,148 740,100" fill="none" stroke="#D4A04A" stroke-width="2.5"/>
  <circle cx="600" cy="197" r="3" fill="#D4A04A"/>
  <circle cx="635" cy="193" r="3" fill="#D4A04A"/>
  <circle cx="670" cy="180" r="3" fill="#D4A04A"/>
  <circle cx="705" cy="148" r="3" fill="#D4A04A"/>
  <circle cx="740" cy="100" r="3" fill="#D4A04A"/>

  <!-- Bottom annotation -->
  <rect x="50" y="260" width="700" height="50" rx="6" fill="#F5F5F5" stroke="#6B7B8D" stroke-width="0.5"/>
  <text x="400" y="280" text-anchor="middle" font-size="11" fill="#333" font-weight="bold">Each sub-task introduces ≈3 unstated presuppositions.</text>
  <text x="400" y="298" text-anchor="middle" font-size="11" fill="#6B7B8D">Ambiguity grows as 3ⁿ where n = number of unstated dependencies between sub-tasks.</text>

  <!-- Arrow from left to center -->
  <line x1="180" y1="140" x2="270" y2="140" stroke="#6B7B8D" stroke-width="1" stroke-dasharray="4,3"/>
  <polygon points="270,137 278,140 270,143" fill="#6B7B8D"/>
  <text x="225" y="133" text-anchor="middle" font-size="9" fill="#6B7B8D">+ complexity</text>

  <!-- Arrow from center to chart -->
  <line x1="480" y1="140" x2="540" y2="140" stroke="#6B7B8D" stroke-width="1" stroke-dasharray="4,3"/>
  <polygon points="540,137 548,140 540,143" fill="#6B7B8D"/>
</svg>
Uploading fig1_ambiguity_multiplier.svg…]()

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

![fig2_past_anatomy](https://github.com/user-attachments/assets/cc620a6f-756a-4e37-839d-7d9ac3829b3e)<svg viewBox="0 0 500 480" xmlns="http://www.w3.org/2000/svg" font-family="Arial, Helvetica, sans-serif">
  <rect width="500" height="480" fill="white"/>

  <text x="250" y="28" text-anchor="middle" font-size="15" font-weight="bold" fill="#333">Figure 2 — PAST Framework Anatomy</text>

  <!-- Left bracket -->
  <path d="M 45 60 L 35 60 L 35 370 L 45 370" fill="none" stroke="#6B7B8D" stroke-width="1.5"/>
  <text x="30" y="220" text-anchor="middle" font-size="10" fill="#6B7B8D" transform="rotate(-90,30,220)">Parallel-ambiguous → Sequential-explicit</text>

  <!-- Problem -->
  <rect x="70" y="50" width="300" height="55" rx="6" fill="#4682B4"/>
  <text x="220" y="75" text-anchor="middle" font-size="14" font-weight="bold" fill="white">P — Problem</text>
  <text x="220" y="93" text-anchor="middle" font-size="10" fill="rgba(255,255,255,0.85)">Scopes the reasoning space</text>
  <!-- Role annotation -->
  <text x="390" y="75" font-size="9" fill="#6B7B8D" font-style="italic">What context?</text>

  <!-- Arrow -->
  <line x1="220" y1="105" x2="220" y2="125" stroke="#6B7B8D" stroke-width="2"/>
  <polygon points="215,123 220,132 225,123" fill="#6B7B8D"/>

  <!-- Action -->
  <rect x="70" y="135" width="300" height="55" rx="6" fill="#5A93C4"/>
  <text x="220" y="160" text-anchor="middle" font-size="14" font-weight="bold" fill="white">A — Action</text>
  <text x="220" y="178" text-anchor="middle" font-size="10" fill="rgba(255,255,255,0.85)">Constrains the operation type</text>
  <text x="390" y="160" font-size="9" fill="#6B7B8D" font-style="italic">What verb?</text>

  <!-- Arrow -->
  <line x1="220" y1="190" x2="220" y2="210" stroke="#6B7B8D" stroke-width="2"/>
  <polygon points="215,208 220,217 225,208" fill="#6B7B8D"/>

  <!-- Steps (expanded) -->
  <rect x="70" y="220" width="300" height="90" rx="6" fill="#D4A04A"/>
  <text x="220" y="242" text-anchor="middle" font-size="14" font-weight="bold" fill="white">S — Steps</text>
  <text x="220" y="258" text-anchor="middle" font-size="10" fill="rgba(255,255,255,0.85)">Linearizes the execution plan</text>
  <!-- Sub-boxes for steps -->
  <rect x="85" y="268" width="55" height="30" rx="3" fill="rgba(255,255,255,0.25)" stroke="rgba(255,255,255,0.5)" stroke-width="1"/>
  <text x="112" y="288" text-anchor="middle" font-size="10" fill="white">Step 1</text>
  <rect x="150" y="268" width="55" height="30" rx="3" fill="rgba(255,255,255,0.25)" stroke="rgba(255,255,255,0.5)" stroke-width="1"/>
  <text x="177" y="288" text-anchor="middle" font-size="10" fill="white">Step 2</text>
  <rect x="215" y="268" width="55" height="30" rx="3" fill="rgba(255,255,255,0.25)" stroke="rgba(255,255,255,0.5)" stroke-width="1"/>
  <text x="242" y="288" text-anchor="middle" font-size="10" fill="white">Step 3</text>
  <rect x="280" y="268" width="55" height="30" rx="3" fill="rgba(255,255,255,0.25)" stroke="rgba(255,255,255,0.5)" stroke-width="1"/>
  <text x="307" y="288" text-anchor="middle" font-size="10" fill="white">Step 4</text>
  <!-- Arrows between sub-steps -->
  <line x1="140" y1="283" x2="150" y2="283" stroke="white" stroke-width="1.5"/>
  <line x1="205" y1="283" x2="215" y2="283" stroke="white" stroke-width="1.5"/>
  <line x1="270" y1="283" x2="280" y2="283" stroke="white" stroke-width="1.5"/>
  <text x="390" y="255" font-size="9" fill="#6B7B8D" font-style="italic">What sequence?</text>

  <!-- Arrow -->
  <line x1="220" y1="310" x2="220" y2="330" stroke="#6B7B8D" stroke-width="2"/>
  <polygon points="215,328 220,337 225,328" fill="#6B7B8D"/>

  <!-- Task -->
  <rect x="70" y="340" width="300" height="55" rx="6" fill="#6B7B8D"/>
  <text x="220" y="365" text-anchor="middle" font-size="14" font-weight="bold" fill="white">T — Task</text>
  <text x="220" y="383" text-anchor="middle" font-size="10" fill="rgba(255,255,255,0.85)">Anchors the output contract</text>
  <text x="390" y="365" font-size="9" fill="#6B7B8D" font-style="italic">What deliverable?</text>

  <!-- Bottom note -->
  <text x="250" y="430" text-anchor="middle" font-size="10" fill="#6B7B8D">PAST converts a parallel-ambiguous instruction</text>
  <text x="250" y="445" text-anchor="middle" font-size="10" fill="#6B7B8D">into a sequential-explicit execution plan.</text>
  <text x="250" y="465" text-anchor="middle" font-size="10" font-weight="bold" fill="#D4A04A">Strength: Completeness | Weakness: No reasoning constraint</text>
</svg>


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


![Uploadi<svg viewBox="0 0 500 500" xmlns="http://www.w3.org/2000/svg" font-family="Arial, Helvetica, sans-serif">
  <rect width="500" height="500" fill="white"/>

  <text x="250" y="28" text-anchor="middle" font-size="15" font-weight="bold" fill="#333">Figure 3 — PLFR Framework Anatomy</text>

  <!-- Right bracket -->
  <path d="M 410 50 L 420 50 L 420 400 L 410 400" fill="none" stroke="#6B7B8D" stroke-width="1.5"/>
  <text x="445" y="230" text-anchor="middle" font-size="10" fill="#6B7B8D" transform="rotate(90,445,230)">Reasoning contract (reproducible + auditable)</text>

  <!-- Prompt -->
  <rect x="70" y="50" width="320" height="50" rx="6" fill="#4682B4"/>
  <text x="230" y="73" text-anchor="middle" font-size="14" font-weight="bold" fill="white">P — Prompt</text>
  <text x="230" y="90" text-anchor="middle" font-size="10" fill="rgba(255,255,255,0.85)">Defines the reasoning entry point</text>

  <!-- Arrow -->
  <line x1="230" y1="100" x2="230" y2="118" stroke="#6B7B8D" stroke-width="2"/>
  <polygon points="225,116 230,125 235,116" fill="#6B7B8D"/>

  <!-- Logic (EMPHASIZED - 1.5x height, thicker border) -->
  <rect x="70" y="125" width="320" height="85" rx="6" fill="#D4A04A" stroke="#B8860B" stroke-width="3"/>
  <text x="230" y="152" text-anchor="middle" font-size="16" font-weight="bold" fill="white">L — Logic ★</text>
  <text x="230" y="172" text-anchor="middle" font-size="11" fill="rgba(255,255,255,0.9)">Constrains HOW the model thinks</text>
  <text x="230" y="192" text-anchor="middle" font-size="9" fill="rgba(255,255,255,0.75)">Specifies analytical method, comparison type, reasoning framework</text>
  <!-- Key indicator -->
  <rect x="80" y="135" width="60" height="18" rx="9" fill="rgba(255,255,255,0.25)"/>
  <text x="110" y="148" text-anchor="middle" font-size="8" fill="white" font-weight="bold">KEY LAYER</text>

  <!-- Arrow -->
  <line x1="230" y1="210" x2="230" y2="228" stroke="#6B7B8D" stroke-width="2"/>
  <polygon points="225,226 230,235 235,226" fill="#6B7B8D"/>

  <!-- Format -->
  <rect x="70" y="235" width="320" height="50" rx="6" fill="#A8C4D8"/>
  <text x="230" y="258" text-anchor="middle" font-size="14" font-weight="bold" fill="#333">F — Format</text>
  <text x="230" y="275" text-anchor="middle" font-size="10" fill="#555">Enforces output predictability</text>

  <!-- Arrow -->
  <line x1="230" y1="285" x2="230" y2="303" stroke="#6B7B8D" stroke-width="2"/>
  <polygon points="225,301 230,310 235,301" fill="#6B7B8D"/>

  <!-- Result -->
  <rect x="70" y="310" width="320" height="50" rx="6" fill="#6B7B8D"/>
  <text x="230" y="333" text-anchor="middle" font-size="14" font-weight="bold" fill="white">R — Result</text>
  <text x="230" y="350" text-anchor="middle" font-size="10" fill="rgba(255,255,255,0.85)">Establishes validation benchmark</text>

  <!-- Feedback arrow from Result back to Logic -->
  <path d="M 70 335 L 45 335 L 45 165 L 70 165" fill="none" stroke="#D4A04A" stroke-width="1.5" stroke-dasharray="5,3"/>
  <polygon points="68,160 60,165 68,170" fill="#D4A04A"/>
  <text x="22" y="250" text-anchor="middle" font-size="9" fill="#D4A04A" transform="rotate(-90,22,250)">Self-validation feedback</text>

  <!-- Bottom note -->
  <text x="250" y="400" text-anchor="middle" font-size="10" fill="#6B7B8D">PLFR contracts the reasoning chain,</text>
  <text x="250" y="415" text-anchor="middle" font-size="10" fill="#6B7B8D">making outputs reproducible and auditable.</text>
  <text x="250" y="440" text-anchor="middle" font-size="10" font-weight="bold" fill="#4682B4">Strength: Reproducibility | Weakness: Assumes task already scoped</text>

  <!-- Comparison callout -->
  <rect x="100" y="458" width="300" height="28" rx="4" fill="#F5F5F5" stroke="#6B7B8D" stroke-width="0.5"/>
  <text x="250" y="477" text-anchor="middle" font-size="9" fill="#6B7B8D">Compare with Figure 2: PAST constrains WHAT → PLFR constrains HOW</text>
</svg>
ng fig3_plfr_anatomy.svg…]()

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


![Uploading<svg viewBox="0 0 700 380" xmlns="http://www.w3.org/2000/svg" font-family="Arial, Helvetica, sans-serif">
  <rect width="700" height="380" fill="white"/>

  <text x="350" y="25" text-anchor="middle" font-size="15" font-weight="bold" fill="#333">Figure 4 — PAST vs. PLFR Decision Landscape</text>

  <!-- LEFT: PAST -->
  <rect x="40" y="50" width="250" height="180" rx="8" fill="#4682B4" opacity="0.08" stroke="#4682B4" stroke-width="2"/>
  <text x="165" y="75" text-anchor="middle" font-size="16" font-weight="bold" fill="#4682B4">PAST</text>

  <!-- Spine/skeleton icon (abstract) -->
  <line x1="165" y1="88" x2="165" y2="155" stroke="#4682B4" stroke-width="3"/>
  <line x1="130" y1="100" x2="200" y2="100" stroke="#4682B4" stroke-width="2"/>
  <line x1="135" y1="120" x2="195" y2="120" stroke="#4682B4" stroke-width="2"/>
  <line x1="140" y1="140" x2="190" y2="140" stroke="#4682B4" stroke-width="2"/>
  <circle cx="165" cy="88" r="5" fill="#4682B4"/>

  <text x="165" y="175" text-anchor="middle" font-size="10" fill="#4682B4" font-weight="bold">Linearizes execution</text>
  <text x="165" y="192" text-anchor="middle" font-size="9" fill="#6B7B8D">Risk: step omission</text>
  <text x="165" y="206" text-anchor="middle" font-size="9" fill="#6B7B8D">Best for: procedural tasks</text>
  <text x="165" y="220" text-anchor="middle" font-size="9" fill="#6B7B8D">Constrains: WHAT to do</text>

  <!-- RIGHT: PLFR -->
  <rect x="410" y="50" width="250" height="180" rx="8" fill="#D4A04A" opacity="0.08" stroke="#D4A04A" stroke-width="2"/>
  <text x="535" y="75" text-anchor="middle" font-size="16" font-weight="bold" fill="#D4A04A">PLFR</text>

  <!-- Brain/network icon (abstract) -->
  <circle cx="520" cy="115" r="8" fill="none" stroke="#D4A04A" stroke-width="1.5"/>
  <circle cx="550" cy="100" r="8" fill="none" stroke="#D4A04A" stroke-width="1.5"/>
  <circle cx="545" cy="135" r="8" fill="none" stroke="#D4A04A" stroke-width="1.5"/>
  <circle cx="510" cy="145" r="8" fill="none" stroke="#D4A04A" stroke-width="1.5"/>
  <circle cx="555" cy="120" r="5" fill="#D4A04A" opacity="0.5"/>
  <line x1="525" y1="110" x2="545" y2="105" stroke="#D4A04A" stroke-width="1" opacity="0.6"/>
  <line x1="545" y1="105" x2="545" y2="130" stroke="#D4A04A" stroke-width="1" opacity="0.6"/>
  <line x1="520" y1="120" x2="515" y2="140" stroke="#D4A04A" stroke-width="1" opacity="0.6"/>
  <line x1="520" y1="120" x2="540" y2="130" stroke="#D4A04A" stroke-width="1" opacity="0.6"/>
  <line x1="545" y1="130" x2="515" y2="140" stroke="#D4A04A" stroke-width="1" opacity="0.6"/>

  <text x="535" y="175" text-anchor="middle" font-size="10" fill="#D4A04A" font-weight="bold">Contracts reasoning</text>
  <text x="535" y="192" text-anchor="middle" font-size="9" fill="#6B7B8D">Risk: wrong method</text>
  <text x="535" y="206" text-anchor="middle" font-size="9" fill="#6B7B8D">Best for: analytical tasks</text>
  <text x="535" y="220" text-anchor="middle" font-size="9" fill="#6B7B8D">Constrains: HOW to think</text>

  <!-- CENTER: Merge zone -->
  <line x1="290" y1="140" x2="330" y2="265" stroke="#6B7B8D" stroke-width="1.5"/>
  <line x1="410" y1="140" x2="370" y2="265" stroke="#6B7B8D" stroke-width="1.5"/>

  <!-- Combined element -->
  <rect x="270" y="248" width="160" height="50" rx="6" fill="#6B7B8D"/>
  <text x="350" y="268" text-anchor="middle" font-size="10" font-weight="bold" fill="white">Layered Architecture</text>
  <text x="350" y="284" text-anchor="middle" font-size="9" fill="rgba(255,255,255,0.85)">PAST decomposes → PLFR executes</text>

  <!-- Decision spectrum -->
  <rect x="70" y="320" width="560" height="40" rx="6" fill="#F5F5F5" stroke="#6B7B8D" stroke-width="0.5"/>

  <!-- Gradient bar -->
  <defs>
    <linearGradient id="spectrum" x1="0" x2="1">
      <stop offset="0%" stop-color="#4682B4"/>
      <stop offset="50%" stop-color="#6B7B8D"/>
      <stop offset="100%" stop-color="#D4A04A"/>
    </linearGradient>
  </defs>
  <rect x="90" y="330" width="520" height="8" rx="4" fill="url(#spectrum)"/>

  <!-- Spectrum labels -->
  <text x="90" y="355" text-anchor="start" font-size="8" fill="#4682B4" font-weight="bold">Procedural / multi-step</text>
  <text x="350" y="355" text-anchor="middle" font-size="8" fill="#6B7B8D" font-weight="bold">Complex agentic: USE BOTH</text>
  <text x="610" y="355" text-anchor="end" font-size="8" fill="#D4A04A" font-weight="bold">Analytical / method-sensitive</text>

  <!-- Spectrum markers -->
  <circle cx="90" cy="334" r="5" fill="#4682B4"/>
  <circle cx="350" cy="334" r="6" fill="#6B7B8D" stroke="white" stroke-width="1.5"/>
  <circle cx="610" cy="334" r="5" fill="#D4A04A"/>
</svg>
 fig4_decision_landscape.svg…]()

![Figure 4 — PAST vs. PLFR Decision Landscape](figures/fig4_decision_landscape.svg)

*Figure 4: PAST (left) operates as structural scaffolding — it constrains what the model does. PLFR (right) operates as a reasoning network — it constrains how the model thinks. For complex agentic tasks (center of spectrum), both frameworks are required in a layered architecture.*

### The Complementary Architecture

In production agentic systems, PAST and PLFR are not alternatives — they are layers. The planning agent uses PAST to decompose a user request into sub-tasks. Each sub-task is dispatched to an execution agent using a PLFR-formatted instruction that specifies the reasoning method and output contract.


![Uploading fig5_two_l<svg viewBox="0 0 700 520" xmlns="http://www.w3.org/2000/svg" font-family="Arial, Helvetica, sans-serif">
  <rect width="700" height="520" fill="white"/>

  <text x="350" y="25" text-anchor="middle" font-size="15" font-weight="bold" fill="#333">Figure 5 — Two-Layer Agentic Architecture (PAST + PLFR)</text>

  <!-- User Request -->
  <rect x="220" y="42" width="260" height="36" rx="18" fill="#E0E0E0" stroke="#999" stroke-width="1"/>
  <text x="350" y="65" text-anchor="middle" font-size="11" fill="#555">User Request (unstructured natural language)</text>

  <!-- Arrow down -->
  <line x1="350" y1="78" x2="350" y2="98" stroke="#6B7B8D" stroke-width="2"/>
  <polygon points="345,96 350,105 355,96" fill="#6B7B8D"/>

  <!-- PLANNING LAYER -->
  <rect x="60" y="108" width="580" height="120" rx="8" fill="#4682B4" opacity="0.1" stroke="#4682B4" stroke-width="2"/>
  <text x="80" y="128" font-size="12" font-weight="bold" fill="#4682B4">PLANNING LAYER</text>

  <!-- Planning Agent box -->
  <rect x="100" y="138" width="500" height="75" rx="6" fill="#4682B4"/>
  <text x="350" y="157" text-anchor="middle" font-size="13" font-weight="bold" fill="white">Planning Agent — PAST Framework</text>

  <!-- PAST components inline -->
  <rect x="120" y="167" width="90" height="30" rx="4" fill="rgba(255,255,255,0.2)" stroke="rgba(255,255,255,0.5)"/>
  <text x="165" y="186" text-anchor="middle" font-size="10" fill="white">Problem</text>
  <text x="220" y="186" font-size="14" fill="white">→</text>
  <rect x="233" y="167" width="80" height="30" rx="4" fill="rgba(255,255,255,0.2)" stroke="rgba(255,255,255,0.5)"/>
  <text x="273" y="186" text-anchor="middle" font-size="10" fill="white">Action</text>
  <text x="323" y="186" font-size="14" fill="white">→</text>
  <rect x="338" y="167" width="70" height="30" rx="4" fill="rgba(255,255,255,0.3)" stroke="rgba(255,255,255,0.7)"/>
  <text x="373" y="186" text-anchor="middle" font-size="10" fill="white" font-weight="bold">Steps</text>
  <text x="418" y="186" font-size="14" fill="white">→</text>
  <rect x="433" y="167" width="70" height="30" rx="4" fill="rgba(255,255,255,0.2)" stroke="rgba(255,255,255,0.5)"/>
  <text x="468" y="186" text-anchor="middle" font-size="10" fill="white">Task</text>

  <!-- Dispatch arrows -->
  <text x="350" y="248" text-anchor="middle" font-size="9" fill="#6B7B8D" font-style="italic">Sub-task dispatch (PLFR envelope)</text>

  <line x1="200" y1="228" x2="140" y2="280" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="137,275 137,284 145,280" fill="#6B7B8D"/>
  <line x1="350" y1="228" x2="350" y2="280" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="345,278 350,287 355,278" fill="#6B7B8D"/>
  <line x1="500" y1="228" x2="560" y2="280" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="555,276 563,284 563,276" fill="#6B7B8D"/>

  <!-- EXECUTION LAYER -->
  <rect x="60" y="270" width="580" height="155" rx="8" fill="#D4A04A" opacity="0.08" stroke="#D4A04A" stroke-width="2"/>
  <text x="80" y="290" font-size="12" font-weight="bold" fill="#D4A04A">EXECUTION LAYER</text>

  <!-- Agent A -->
  <rect x="75" y="300" width="160" height="110" rx="6" fill="#D4A04A"/>
  <text x="155" y="320" text-anchor="middle" font-size="11" font-weight="bold" fill="white">Execution Agent A</text>
  <text x="155" y="334" text-anchor="middle" font-size="9" fill="rgba(255,255,255,0.8)">PLFR Contract</text>
  <rect x="88" y="342" width="134" height="18" rx="3" fill="rgba(255,255,255,0.15)"/>
  <text x="155" y="355" text-anchor="middle" font-size="8" fill="white">Prompt | Logic | Format | Result</text>
  <rect x="88" y="366" width="134" height="30" rx="3" fill="rgba(255,255,255,0.1)"/>
  <text x="155" y="380" text-anchor="middle" font-size="8" fill="rgba(255,255,255,0.7)">Logic: delta-based</text>
  <text x="155" y="392" text-anchor="middle" font-size="8" fill="rgba(255,255,255,0.7)">comparison method</text>

  <!-- Agent B -->
  <rect x="270" y="300" width="160" height="110" rx="6" fill="#D4A04A" opacity="0.9"/>
  <text x="350" y="320" text-anchor="middle" font-size="11" font-weight="bold" fill="white">Execution Agent B</text>
  <text x="350" y="334" text-anchor="middle" font-size="9" fill="rgba(255,255,255,0.8)">PLFR Contract</text>
  <rect x="283" y="342" width="134" height="18" rx="3" fill="rgba(255,255,255,0.15)"/>
  <text x="350" y="355" text-anchor="middle" font-size="8" fill="white">Prompt | Logic | Format | Result</text>
  <rect x="283" y="366" width="134" height="30" rx="3" fill="rgba(255,255,255,0.1)"/>
  <text x="350" y="380" text-anchor="middle" font-size="8" fill="rgba(255,255,255,0.7)">Logic: NPS correlation</text>
  <text x="350" y="392" text-anchor="middle" font-size="8" fill="rgba(255,255,255,0.7)">on flagged tiers only</text>

  <!-- Agent C -->
  <rect x="465" y="300" width="160" height="110" rx="6" fill="#D4A04A" opacity="0.8"/>
  <text x="545" y="320" text-anchor="middle" font-size="11" font-weight="bold" fill="white">Execution Agent C</text>
  <text x="545" y="334" text-anchor="middle" font-size="9" fill="rgba(255,255,255,0.8)">PLFR Contract</text>
  <rect x="478" y="342" width="134" height="18" rx="3" fill="rgba(255,255,255,0.15)"/>
  <text x="545" y="355" text-anchor="middle" font-size="8" fill="white">Prompt | Logic | Format | Result</text>
  <rect x="478" y="366" width="134" height="30" rx="3" fill="rgba(255,255,255,0.1)"/>
  <text x="545" y="380" text-anchor="middle" font-size="8" fill="rgba(255,255,255,0.7)">Logic: cite specific data</text>
  <text x="545" y="392" text-anchor="middle" font-size="8" fill="rgba(255,255,255,0.7)">per recommendation</text>

  <!-- Convergence arrows -->
  <line x1="155" y1="410" x2="300" y2="455" stroke="#6B7B8D" stroke-width="1.5"/>
  <line x1="350" y1="410" x2="350" y2="455" stroke="#6B7B8D" stroke-width="1.5"/>
  <line x1="545" y1="410" x2="400" y2="455" stroke="#6B7B8D" stroke-width="1.5"/>

  <!-- Assembled Output -->
  <rect x="250" y="458" width="200" height="36" rx="6" fill="#6B7B8D"/>
  <text x="350" y="481" text-anchor="middle" font-size="12" font-weight="bold" fill="white">Assembled Output</text>

  <!-- Layer labels on right -->
  <rect x="650" y="140" width="4" height="70" fill="#4682B4"/>
  <text x="660" y="180" font-size="8" fill="#4682B4">PAST</text>
  <rect x="650" y="300" width="4" height="110" fill="#D4A04A"/>
  <text x="660" y="360" font-size="8" fill="#D4A04A">PLFR</text>
</svg>
ayer_architecture.svg…]()

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


![Uploading <svg viewBox="0 0 750 550" xmlns="http://www.w3.org/2000/svg" font-family="Arial, Helvetica, sans-serif">
  <rect width="750" height="550" fill="white"/>

  <text x="375" y="25" text-anchor="middle" font-size="15" font-weight="bold" fill="#333">Figure 6 — "Confidently Structured, Silently Wrong" Failure</text>

  <!-- Data Source -->
  <rect x="225" y="40" width="300" height="100" rx="6" fill="#F5F5F5" stroke="#999" stroke-width="1.5"/>
  <text x="375" y="58" text-anchor="middle" font-size="12" font-weight="bold" fill="#333">Churn Data (Q2 vs. Q3)</text>

  <!-- Mini data table -->
  <text x="260" y="76" font-size="9" fill="#555" font-weight="bold">Tier</text>
  <text x="340" y="76" font-size="9" fill="#555" font-weight="bold">Q2</text>
  <text x="400" y="76" font-size="9" fill="#555" font-weight="bold">Q3</text>
  <text x="470" y="76" font-size="9" fill="#555" font-weight="bold">Delta</text>

  <line x1="245" y1="80" x2="510" y2="80" stroke="#CCC" stroke-width="0.5"/>

  <text x="260" y="94" font-size="9" fill="#333">Free</text>
  <text x="340" y="94" font-size="9" fill="#333">3.5%</text>
  <text x="400" y="94" font-size="9" fill="#333">9.0%</text>
  <text x="470" y="94" font-size="9" fill="#D4A04A" font-weight="bold">+5.5pp</text>

  <text x="260" y="108" font-size="9" fill="#333">Pro</text>
  <text x="340" y="108" font-size="9" fill="#333">4.8%</text>
  <text x="400" y="108" font-size="9" fill="#333">5.5%</text>
  <text x="470" y="108" font-size="9" fill="#333">+0.7pp</text>

  <text x="260" y="122" font-size="9" fill="#333">Enterprise</text>
  <text x="340" y="122" font-size="9" fill="#333">11.5%</text>
  <text x="400" y="122" font-size="9" fill="#333">12.0%</text>
  <text x="470" y="122" font-size="9" fill="#333">+0.5pp</text>

  <!-- Fork arrows -->
  <line x1="300" y1="140" x2="175" y2="180" stroke="#D4A04A" stroke-width="2"/>
  <polygon points="172,175 172,184 180,180" fill="#D4A04A"/>
  <line x1="450" y1="140" x2="575" y2="180" stroke="#4682B4" stroke-width="2"/>
  <polygon points="570,176 578,180 578,184" fill="#4682B4"/>

  <!-- LEFT PIPELINE: PAST Only -->
  <rect x="30" y="185" width="290" height="280" rx="8" fill="none" stroke="#D4A04A" stroke-width="2.5" stroke-dasharray="0"/>
  <text x="175" y="205" text-anchor="middle" font-size="13" font-weight="bold" fill="#D4A04A">PAST Only</text>

  <rect x="55" y="215" width="240" height="40" rx="4" fill="#D4A04A" opacity="0.15"/>
  <text x="175" y="233" text-anchor="middle" font-size="10" fill="#333">Step 3: "Identify the tier with the</text>
  <text x="175" y="247" text-anchor="middle" font-size="10" fill="#333" font-weight="bold">highest churn"</text>

  <!-- Arrow -->
  <line x1="175" y1="255" x2="175" y2="275" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="170,273 175,282 180,273" fill="#6B7B8D"/>

  <!-- Method box -->
  <rect x="55" y="282" width="240" height="30" rx="4" fill="#FFF3E0" stroke="#D4A04A" stroke-width="1"/>
  <text x="175" y="302" text-anchor="middle" font-size="10" fill="#8B6914">Method: Absolute rate (model's default)</text>

  <!-- Arrow -->
  <line x1="175" y1="312" x2="175" y2="330" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="170,328 175,337 180,328" fill="#6B7B8D"/>

  <!-- Result -->
  <rect x="55" y="337" width="240" height="40" rx="4" fill="#FFEBEE" stroke="#C62828" stroke-width="1.5"/>
  <text x="175" y="355" text-anchor="middle" font-size="12" font-weight="bold" fill="#C62828">Enterprise (12.0%)</text>
  <text x="175" y="370" text-anchor="middle" font-size="9" fill="#C62828">Highest absolute rate</text>

  <!-- Red X indicator (shape, not just color) -->
  <circle cx="55" cy="357" r="12" fill="#C62828"/>
  <text x="55" y="362" text-anchor="middle" font-size="14" font-weight="bold" fill="white">✗</text>

  <!-- Arrow -->
  <line x1="175" y1="377" x2="175" y2="395" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="170,393 175,402 180,393" fill="#6B7B8D"/>

  <!-- Business decision -->
  <rect x="55" y="402" width="240" height="45" rx="4" fill="#C62828" opacity="0.1" stroke="#C62828" stroke-width="1"/>
  <text x="175" y="420" text-anchor="middle" font-size="10" fill="#C62828" font-weight="bold">Business Decision:</text>
  <text x="175" y="436" text-anchor="middle" font-size="10" fill="#C62828">"Focus retention on Enterprise"</text>


  <!-- RIGHT PIPELINE: PAST + PLFR -->
  <rect x="430" y="185" width="290" height="280" rx="8" fill="none" stroke="#4682B4" stroke-width="2.5"/>
  <text x="575" y="205" text-anchor="middle" font-size="13" font-weight="bold" fill="#4682B4">PAST + PLFR</text>

  <rect x="455" y="215" width="240" height="40" rx="4" fill="#4682B4" opacity="0.1"/>
  <text x="575" y="233" text-anchor="middle" font-size="10" fill="#333">Logic: "Use absolute delta,</text>
  <text x="575" y="247" text-anchor="middle" font-size="10" fill="#333" font-weight="bold">not absolute level"</text>

  <!-- Arrow -->
  <line x1="575" y1="255" x2="575" y2="275" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="570,273 575,282 580,273" fill="#6B7B8D"/>

  <!-- Method box -->
  <rect x="455" y="282" width="240" height="30" rx="4" fill="#E3F2FD" stroke="#4682B4" stroke-width="1"/>
  <text x="575" y="302" text-anchor="middle" font-size="10" fill="#1565C0">Method: Delta comparison (specified)</text>

  <!-- Arrow -->
  <line x1="575" y1="312" x2="575" y2="330" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="570,328 575,337 580,328" fill="#6B7B8D"/>

  <!-- Result -->
  <rect x="455" y="337" width="240" height="40" rx="4" fill="#E8F5E9" stroke="#2E7D32" stroke-width="1.5"/>
  <text x="575" y="355" text-anchor="middle" font-size="12" font-weight="bold" fill="#2E7D32">Free (+5.5pp)</text>
  <text x="575" y="370" text-anchor="middle" font-size="9" fill="#2E7D32">Highest churn increase</text>

  <!-- Green check indicator (shape, not just color) -->
  <circle cx="455" cy="357" r="12" fill="#2E7D32"/>
  <text x="455" y="362" text-anchor="middle" font-size="14" font-weight="bold" fill="white">✓</text>

  <!-- Arrow -->
  <line x1="575" y1="377" x2="575" y2="395" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="570,393 575,402 580,393" fill="#6B7B8D"/>

  <!-- Business decision -->
  <rect x="455" y="402" width="240" height="45" rx="4" fill="#2E7D32" opacity="0.1" stroke="#2E7D32" stroke-width="1"/>
  <text x="575" y="420" text-anchor="middle" font-size="10" fill="#2E7D32" font-weight="bold">Business Decision:</text>
  <text x="575" y="436" text-anchor="middle" font-size="10" fill="#2E7D32">"Investigate Free-tier changes"</text>

  <!-- Bottom callout -->
  <rect x="125" y="485" width="500" height="45" rx="6" fill="#333" opacity="0.05" stroke="#333" stroke-width="1"/>
  <text x="375" y="505" text-anchor="middle" font-size="12" font-weight="bold" fill="#333">Same data. Same model. Opposite decisions.</text>
  <text x="375" y="522" text-anchor="middle" font-size="11" fill="#6B7B8D">The architecture — not the model — is the difference.</text>
</svg>
fig6_failure_mode.svg…]()

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


![Uploadi<svg viewBox="0 0 800 420" xmlns="http://www.w3.org/2000/svg" font-family="Arial, Helvetica, sans-serif">
  <rect width="800" height="420" fill="white"/>

  <text x="400" y="25" text-anchor="middle" font-size="15" font-weight="bold" fill="#333">Figure 7 — Pipeline Topology: Human Decision Node</text>

  <!-- LEFT PANEL: AI Proposed (Parallel) -->
  <rect x="20" y="45" width="300" height="300" rx="8" fill="none" stroke="#D4A04A" stroke-width="2" stroke-dasharray="8,4"/>
  <text x="170" y="68" text-anchor="middle" font-size="12" font-weight="bold" fill="#D4A04A">AI Proposed: Parallel</text>

  <!-- Step 1 -->
  <rect x="105" y="82" width="130" height="35" rx="5" fill="#4682B4"/>
  <text x="170" y="104" text-anchor="middle" font-size="10" fill="white">Step 1: Extract data</text>

  <!-- Fork arrows from Step 1 -->
  <line x1="140" y1="117" x2="95" y2="155" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="92,150 92,159 100,155" fill="#6B7B8D"/>
  <line x1="200" y1="117" x2="245" y2="155" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="240,151 248,155 248,159" fill="#6B7B8D"/>

  <!-- Step 2 (parallel) -->
  <rect x="35" y="160" width="120" height="35" rx="5" fill="#D4A04A"/>
  <text x="95" y="182" text-anchor="middle" font-size="9" fill="white">Step 2: Compute deltas</text>

  <!-- Step 3 (parallel) -->
  <rect x="185" y="160" width="120" height="35" rx="5" fill="#D4A04A"/>
  <text x="245" y="182" text-anchor="middle" font-size="9" fill="white">Step 3: NPS cross-ref</text>

  <!-- Parallel indicator -->
  <text x="170" y="180" text-anchor="middle" font-size="20" fill="#D4A04A">‖</text>

  <!-- Join arrows to Step 4 -->
  <line x1="95" y1="195" x2="140" y2="235" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="137,230 137,239 145,235" fill="#6B7B8D"/>
  <line x1="245" y1="195" x2="200" y2="235" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="195,231 203,235 203,239" fill="#6B7B8D"/>

  <!-- Step 4 -->
  <rect x="105" y="240" width="130" height="35" rx="5" fill="#6B7B8D"/>
  <text x="170" y="262" text-anchor="middle" font-size="10" fill="white">Step 4: Recommend</text>

  <!-- Problem annotation -->
  <rect x="40" y="290" width="260" height="40" rx="4" fill="#FFEBEE" stroke="#C62828" stroke-width="1"/>
  <text x="170" y="307" text-anchor="middle" font-size="9" fill="#C62828" font-weight="bold">⚠ Step 3 runs WITHOUT flag data</text>
  <text x="170" y="322" text-anchor="middle" font-size="8" fill="#C62828">Must cross-reference ALL tiers (wasteful)</text>

  <!-- CENTRAL: Human Decision Node -->
  <g transform="translate(370, 185)">
    <!-- Diamond -->
    <polygon points="0,-40 50,0 0,40 -50,0" fill="#4682B4" stroke="#2C5F8A" stroke-width="2"/>
    <text x="0" y="-8" text-anchor="middle" font-size="8" fill="white" font-weight="bold">HUMAN</text>
    <text x="0" y="5" text-anchor="middle" font-size="8" fill="white" font-weight="bold">DECISION</text>
    <text x="0" y="18" text-anchor="middle" font-size="8" fill="white" font-weight="bold">NODE</text>
  </g>

  <!-- Arrows into and out of diamond -->
  <line x1="320" y1="185" x2="320" y2="185" stroke="#6B7B8D" stroke-width="1.5"/>
  <line x1="420" y1="185" x2="475" y2="185" stroke="#4682B4" stroke-width="2"/>
  <polygon points="473,180 482,185 473,190" fill="#4682B4"/>

  <!-- Annotation below diamond -->
  <text x="370" y="240" text-anchor="middle" font-size="9" fill="#4682B4" font-style="italic">Hidden dependency</text>
  <text x="370" y="253" text-anchor="middle" font-size="9" fill="#4682B4" font-style="italic">detected → topology</text>
  <text x="370" y="266" text-anchor="middle" font-size="9" fill="#4682B4" font-style="italic">changed</text>

  <!-- RIGHT PANEL: Human Override (Sequential) -->
  <rect x="480" y="45" width="300" height="300" rx="8" fill="none" stroke="#4682B4" stroke-width="2.5"/>
  <text x="630" y="68" text-anchor="middle" font-size="12" font-weight="bold" fill="#4682B4">Human Override: Sequential</text>

  <!-- Step 1 -->
  <rect x="565" y="82" width="130" height="35" rx="5" fill="#4682B4"/>
  <text x="630" y="104" text-anchor="middle" font-size="10" fill="white">Step 1: Extract data</text>

  <!-- Arrow -->
  <line x1="630" y1="117" x2="630" y2="150" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="625,148 630,157 635,148" fill="#6B7B8D"/>

  <!-- Step 2 -->
  <rect x="565" y="160" width="130" height="35" rx="5" fill="#D4A04A"/>
  <text x="630" y="182" text-anchor="middle" font-size="9" fill="white">Step 2: Compute deltas</text>

  <!-- HIGHLIGHTED dependency arrow -->
  <line x1="630" y1="195" x2="630" y2="225" stroke="#2E7D32" stroke-width="3"/>
  <polygon points="624,223 630,234 636,223" fill="#2E7D32"/>
  <!-- Dependency label -->
  <rect x="648" y="200" width="120" height="26" rx="3" fill="#E8F5E9" stroke="#2E7D32" stroke-width="1"/>
  <text x="708" y="212" text-anchor="middle" font-size="7.5" fill="#2E7D32" font-weight="bold">Step 3 needs flagged</text>
  <text x="708" y="222" text-anchor="middle" font-size="7.5" fill="#2E7D32" font-weight="bold">tiers from Step 2</text>

  <!-- Step 3 -->
  <rect x="565" y="238" width="130" height="35" rx="5" fill="#D4A04A"/>
  <text x="630" y="260" text-anchor="middle" font-size="9" fill="white">Step 3: NPS cross-ref</text>

  <!-- Arrow -->
  <line x1="630" y1="273" x2="630" y2="300" stroke="#6B7B8D" stroke-width="1.5"/>
  <polygon points="625,298 630,307 635,298" fill="#6B7B8D"/>

  <!-- Step 4 -->
  <rect x="565" y="310" width="130" height="35" rx="5" fill="#6B7B8D"/>
  <text x="630" y="332" text-anchor="middle" font-size="10" fill="white">Step 4: Recommend</text>

  <!-- Bottom annotation -->
  <rect x="150" y="370" width="500" height="40" rx="6" fill="#F5F5F5" stroke="#6B7B8D" stroke-width="0.5"/>
  <text x="400" y="388" text-anchor="middle" font-size="10" fill="#333" font-weight="bold">The AI parsed dependencies syntactically (field matching).</text>
  <text x="400" y="403" text-anchor="middle" font-size="10" fill="#6B7B8D">The human traced dependencies semantically ("flagged tiers" → Step 2 output).</text>
</svg>
ng fig7_human_decision_node.svg…]()

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
