# Chapter 8: PAST, PLFR, and Structured Prompt Frameworks
**Structured Decomposition as Architectural Scaffolding**
From the book *Design of Agentic Systems with Case Studies*

## 🎯 Core Claim

Structured prompt frameworks — PAST (Problem, Action, Steps, Task) and PLFR (Prompt, Logic, Format, Result) — impose logical scaffolding that reduces ambiguity and improves output reliability at scale. The layered combination of PAST for task decomposition and PLFR for reasoning contracts prevents the "confidently structured, silently wrong" failure where correctly formatted outputs silently apply the wrong analytical method.

**Architecture is the leverage point, not the model.**

---

## 🏗️ Architecture

### Two-Layer Agentic Architecture

The system layers two complementary frameworks — PAST at the planning layer and PLFR at the execution layer:

![Two-Layer Agentic Architecture](figures/fig5_two_layer_architecture.svg)

Each execution agent receives a PLFR contract — it cannot choose its own analytical method. The **Logic** component (★) is the architectural primitive that prevents method non-determinism.

### PAST Framework — Linearizes Execution

![PAST Framework Anatomy](figures/fig2_past_anatomy.svg)

PAST converts parallel-ambiguous instructions into sequential-explicit plans. Its strength is completeness; its weakness is that it constrains *what* the model does without constraining *how* it reasons.

### PLFR Framework — Contracts Reasoning

![PLFR Framework Anatomy](figures/fig3_plfr_anatomy.svg)

PLFR specifies the analytical method, output schema, and validation criteria. The Logic layer is architecturally dominant — it is what distinguishes PLFR from a simple format template. The feedback arrow from Result back to Logic enables self-validation.

### The Architecture-Model Boundary

| Layer | Responsibility | Who Controls It |
|-------|---------------|-----------------|
| **PAST** | Task decomposition, sequencing, scope | Prompt architect (human) |
| **PLFR Logic** | Analytical method, reasoning constraints | Prompt architect (human) |
| **PLFR Format** | Output schema, structure | Prompt architect (human) |
| **LLM** | Language understanding, generation, execution | Model |

The LLM executes the architecture. It does not design it.

### When to Use PAST vs. PLFR

![PAST vs. PLFR Decision Landscape](figures/fig4_decision_landscape.svg)

### The Human Decision Node

![Pipeline Topology: Human Decision Node](figures/fig7_human_decision_node.svg)

When the AI scaffold proposes a pipeline topology, the system **halts** and requires explicit human verification of semantic dependencies. In this case, the AI proposed parallel execution for Steps 2 and 3. The human detected a hidden dependency ("flagged tiers" in Step 3 are computed by Step 2) and overrode to sequential execution. This is an architectural primitive, not a safety disclaimer.

---

## ⚡ Quick Start

### 1. Clone & Install
```bash
git clone <repo-url>
cd chapter-08-past-plfr
pip install pandas tabulate
```

### 2. Run the Demo Notebook
```bash
jupyter notebook chapter_08_notebook.ipynb
```

**What you'll see:**
- Synthetic churn dataset designed to expose the failure mode
- PAST-only pipeline returning **Enterprise** as primary churn driver (wrong answer)
- PAST+PLFR pipeline returning **Free** as primary churn driver (correct answer)
- AI scaffold proposing parallel execution → Human Decision Node overriding to sequential
- Full corrected pipeline with PLFR contracts on every agent

### 3. Trigger the Failure Mode
In the notebook, run **Section 6** — remove the Logic component and execute 10 simulated runs:

| # | Failure | What Happens |
|---|---------|-------------|
| 1 | **Missing Logic** | Model non-deterministically chooses between absolute rate, delta, and percentage change |
| 2 | **Wrong Driver** | Enterprise returned instead of Free (correct format, wrong analysis) |
| 3 | **Inconsistent Results** | Same prompt + same data → different answers across runs |
| 4 | **Opposite Business Decisions** | "Focus on Enterprise" vs. "Investigate Free" from identical inputs |

---

## 💥 The Failure Case: "Confidently Structured, Silently Wrong"

### The Ambiguity Multiplier

![The Ambiguity Multiplier](figures/fig1_ambiguity_multiplier.svg)

Ambiguity grows exponentially (≈3ⁿ) with the number of unstated dependencies between sub-tasks. Structured frameworks collapse this space before the model encounters it.

### The Failure — Same Data, Opposite Decisions

![Confidently Structured, Silently Wrong](figures/fig6_failure_mode.svg)

When you remove the PLFR Logic component and use PAST-only, the model defaults to absolute rate comparison and identifies **Enterprise** (12.0% — stable, no change) as the primary churn driver. With PLFR's Logic component specifying delta-based comparison, the model correctly identifies **Free** (+5.5pp — massive spike) as the driver.

The model was correct both times. The architecture was different.

---

## 📊 Comparison

| Approach | Sequence | Method | Format | Reproducible? |
|----------|----------|--------|--------|---------------|
| **Unstructured** | ❌ Ambiguous | ❌ Unconstrained | ❌ Variable | No |
| **PAST Only** | ✅ Linearized | ❌ Unconstrained | ✅ Specified | Partially |
| **PAST + PLFR** | ✅ Linearized | ✅ Locked | ✅ Specified | **Yes** |

---

## 📁 Repository Structure

```
├── chapter_08_PAST_PLFR.md          # Full chapter prose with 7 embedded figures (~620 lines)
├── chapter_08_notebook.ipynb         # Runnable demo — failure case, AI scaffold, full pipeline
├── authors_note.docx                # 3-page author's note (design choices, tool usage, self-assessment)
├── video_storyboard.md              # 10-min video script (Explain → Show → Try)
├── README.md                        # This file
└── figures/
    ├── fig1_ambiguity_multiplier.svg    # Exponential 3ⁿ branching + inset chart
    ├── fig2_past_anatomy.svg            # 4-layer PAST pipeline with expanded step sub-boxes
    ├── fig3_plfr_anatomy.svg            # PLFR with emphasized Logic layer + feedback arrow
    ├── fig4_decision_landscape.svg      # PAST vs. PLFR decision spectrum with merge zone
    ├── fig5_two_layer_architecture.svg  # Planning (PAST) → Execution (PLFR) system diagram
    ├── fig6_failure_mode.svg            # Side-by-side: same data → opposite business decisions
    └── fig7_human_decision_node.svg     # Parallel → Sequential topology correction
```

---

## 📖 Chapter Outline

| Section | Content |
|---------|---------|
| **8.1 The Scenario** | Fintech team's ad-hoc LLM requests silently degrade as task complexity grows |
| **8.2 The Mechanism** | Ambiguity Multiplier (3ⁿ), PAST linearizes execution, PLFR contracts reasoning |
| **8.3 The Design Decision** | When to use PAST vs. PLFR — decision criteria + complementary layered architecture |
| **8.4 The Failure Case** | "Confidently structured, silently wrong" — Enterprise vs. Free, triggered and observed |
| **8.5 Agentic Pipelines** | PAST as planning backbone, PLFR as agent-to-agent contracts, Human Decision Node |
| **8.6 Exercise** | Remove Logic → run 10 times → observe non-deterministic method selection |
| **8.7 Key Takeaways** | 5 takeaways: ambiguity scaling, linearization, reasoning contracts, layered pattern, failure mode |

---

## 🛠️ Tool Audits Applied

This chapter was iteratively refined using the prescribed pedagogical tools:

| Tool | Rounds | Key Fix |
|------|--------|---------|
| **Bookie the Bookmaker** | 3 rounds | Rejected survey-style framing (PAST/PLFR/RACE/CRISPE taxonomy) → narrowed to architectural argument with complementary failure modes |
| **Eddy the Editor** | 1 full audit | Flagged: (1) ambiguity multiplier described but not mechanistically explained, (2) failure case described as risk not demonstrated, (3) exercise asked reader to "think about" not "trigger" |
| **Figure Architect** | 1 zone analysis | Identified 7 high-assertion zones; generated structural + aesthetic prompts for all 7 figures following colorblind-accessible palette (steel blue / warm amber / slate grey) |
| **Courses** | 1 run | Generated 5 Bloom's-aligned learning outcomes driving prose structure, demo design, and video sequence |

---

## 🔑 Key Design Decisions

- **No API key required** — simulated execution proves the architecture, not the model, is the leverage point
- **Failures are triggered, not described** — Section 6 produces observable non-deterministic output when Logic is removed
- **Human Decision Node is structural** — the pipeline architecturally cannot finalize topology without human verification of semantic dependencies
- **Figures follow Figure Architecture report** — 7 SVGs generated from structural prompts, all colorblind-accessible (blue/amber/grey with shape indicators)
- **PAST and PLFR are layers, not alternatives** — the chapter's architectural argument is their complementary combination, not a framework comparison
- **Coordinator is implicit** — unlike a three-agent network, the PAST planning layer itself serves as the coordination mechanism

---

## 👤 Author

Husain — Northeastern University, M.S. Information Systems

---

## 📄 License

Educational use for *Design of Agentic Systems with Case Studies*.
