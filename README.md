# Chapter 8: PAST, PLFR, and Structured Prompt Frameworks

**Book:** *Design of Agentic Systems with Case Studies*  
**Course:** INFO 7375 — Prompt Engineering for Generative AI  
**Chapter Type:** Type A — Architectural Pattern  
**Author:** Husain

---

## Core Claim

> "After reading this chapter, a student will understand structured prompt frameworks (PAST and PLFR) well enough to design task decomposition architectures for agentic pipelines without making the mistake of relying on unstructured natural language prompts that silently degrade under task complexity."

**Master Argument:** Architecture is the leverage point, not the model. This chapter demonstrates that the structure imposed on instructions determines output quality more than the model chosen to execute them.

---

## Repository Contents

| File | Description |
|------|-------------|
| `chapter_08_PAST_PLFR.md` | Full chapter prose (Substack-ready) |
| `chapter_08_notebook.ipynb` | Runnable demo — failure case, AI scaffold, pipeline |
| `authors_note.docx` | 3-page pedagogical report |
| `video_storyboard.md` | 10-minute Explain → Show → Try video script |
| `README.md` | This file |

---

## Quick Start

```bash
# Clone the repo
git clone <repo-url>
cd chapter-08-past-plfr

# Install dependencies
pip install pandas tabulate

# Run the notebook
jupyter notebook chapter_08_notebook.ipynb
```

No API keys required. The notebook uses simulated execution to demonstrate the failure mode deterministically.

---

## The Failure Mode

**"Confidently Structured, Silently Wrong"**

A PAST-structured prompt produces a correctly formatted, correctly sequenced output that uses the wrong analytical method. The model follows your steps exactly — the steps just don't specify the right reasoning.

**Demonstration:**
- PAST-only prompt → identifies **Enterprise** as primary churn driver (highest absolute rate)
- PAST+PLFR prompt → identifies **Free** as primary churn driver (highest rate *increase*)
- Same data, same model. The business decisions are **opposite**.

**Trigger it yourself:** In Section 6 of the notebook, remove the Logic component from the PLFR contract and run the simulation 10 times. Observe inconsistent results.

---

## Human Decision Node

The AI scaffold proposed parallel execution of pipeline Steps 2 and 3. This was **rejected** because Step 3 references "flagged tiers" — which are computed by Step 2. The AI missed this semantic dependency because it only analyzed syntactic dependencies. The corrected topology is sequential.

This decision is documented in the notebook (Section 4) and demonstrated on camera in the video (Scene 7).

---

## Architectural Pattern

```
PAST (Planning Layer)          PLFR (Execution Layer)
─────────────────────          ──────────────────────
Decomposes complex             Contracts reasoning for
request into sequential        each sub-task with method,
dispatchable steps             format, and validation

     Problem                        Prompt
     Action          ──────►        Logic    ← KEY COMPONENT
     Steps                          Format
     Task                           Result
```

Neither framework alone is sufficient. PAST without PLFR → correctly sequenced, wrong method. PLFR without PAST → correct method, wrong scope.

---

## Video

[YouTube/Vimeo link — unlisted]

**Structure:** Explain (2:30) → Show (5:30) → Try (2:00)

---

## License

Educational use. Chapter content for *Design of Agentic Systems with Case Studies*.
