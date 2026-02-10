# Honest Review: DesignLens Architecture
## Multiple Hats, Multiple Perspectives

---

## HAT 1: THE CRITIC — "Is This Senseless?"

**No, it's not senseless. The modular tool idea is architecturally sound.**

But here's the hard truth:

### You're designing a framework when you should be proving a product.

Phase 1 goal (from your own CLAUDE.md):
> "Prove the concept works — one base model, two schools, distinct outputs."

Building 5 modular plug-and-play tools is a PHASE 2-3 concern.
Phase 1 should be ugly, fast, and prove ONE thing:
**Do Nielsen and Norman adapters actually sound different?**

If the answer is no, the tooling doesn't matter.
If the answer is yes, THEN you modularize.

**Verdict:** The vision is right. The timing is wrong.
Build the ugly version first. Modularize after you've proven distinctiveness.

---

## HAT 2: THE CTO — "Build vs Buy"

Your Tool 1 (Extract + Clean) already exists as a product:

| Your Tool | Already Exists As | Maturity |
|-----------|-------------------|----------|
| Tool 1: Extract | Unstructured.io | Production-grade, 30+ connectors |
| Tool 1: Extract | LlamaIndex data connectors | Mature, open-source |
| Tool 4: RAG | LlamaIndex / LangChain | Industry standard |
| Tool 4: RAG | Haystack | Production-ready |
| Tool 3: FT | Unsloth | Already in your plan |

**The CTO question:** Where is YOUR value?

It's NOT in building another extraction pipeline.
It's NOT in building another RAG framework.

Your moat is:
1. The school-specific knowledge CURATION (what goes into nielsen.txt vs norman.txt)
2. The quality of training pairs (the LLM-as-judge pipeline)
3. The adapter hot-swap UX (compare mode)
4. Riaz's UX domain expertise deciding WHAT to extract and HOW to frame it

**CTO recommendation:**
- Tool 1 (Extract): USE Unstructured.io or LlamaIndex connectors. Don't build.
- Tool 2 (Pair Gen): BUILD THIS. This is your differentiator.
- Tool 3 (FT): USE Unsloth. Don't wrap it unnecessarily.
- Tool 4 (RAG): USE LlamaIndex/Chroma. Don't build.
- Tool 5 (RAFT): BUILD THIS (later). Novel combination of your adapters + RAG.

Spend 80% of your time on Tool 2 (data quality) and 20% on everything else.

---

## HAT 3: THE LLM ENGINEER — "Will This Actually Work?"

### Concern 1: Do you even need fine-tuning?

This is the elephant in the room.

With modern LLMs (Claude, GPT-4, even Llama 3 70B), you can get
"design school" behavior with just:
  - A well-crafted system prompt (you already have these!)
  - RAG with school-specific sources

Test this BEFORE training any adapters:
  1. Take your nielsen.txt system prompt
  2. Send 20 scenarios to Claude/GPT-4 with that prompt
  3. Do the same with norman.txt
  4. Run your distinctiveness eval
  5. If accuracy > 80%, prompting alone works and FT is unnecessary

If prompting alone scores 90%+, your project becomes:
"Curated school prompts + RAG" instead of "Fine-tuned adapters"
That's still valuable! But it changes the architecture completely.

### Concern 2: Gemma 2B is very small

A 2B model fine-tuned with QLoRA on 500 synthetic pairs...
the outputs will likely be:
  - Shorter and less nuanced than GPT-4/Claude
  - More repetitive (small models loop)
  - Possibly losing the subtle differences between schools

This isn't a dealbreaker — it's a POC. But set expectations:
the fine-tuned 2B model won't match prompt-engineered Claude.
The value is in proving the CONCEPT of hot-swappable school adapters.

### Concern 3: Synthetic data quality ceiling

You're using Claude/GPT-4 to generate training data for Gemma 2B.
This means your fine-tuned model can NEVER exceed the quality of
the teacher model's understanding of Nielsen/Norman.

The real question: Is it better to just call Claude with a good
system prompt at inference time?

**When FT beats prompting:**
  - You need offline/local inference
  - You need consistent style without paying per-token
  - You want to deploy on HuggingFace Spaces (free, no API costs)
  - You want community to download and use adapters offline

These are VALID reasons for your project. Just be clear about them.

### Concern 4: RAFT is exciting but premature

RAFT (Retrieval Augmented Fine-Tuning) trains the model to use
retrieved docs and ignore distractors. It's powerful but:
  - Adds significant complexity (need retrieval during training)
  - Requires a mature extraction pipeline first
  - The original RAFT paper showed gains on domain-specific QA,
    but your use case (style/reasoning adaptation) is different

Save RAFT for Phase 3. Focus Phase 1 on proving FT works at all.

---

## HAT 4: THE AI ENGINEER — "What Would I Actually Build?"

### The Pragmatic Phase 1 Architecture

Forget 5 modular tools. Build 3 scripts:

```
Script 1: generate_pairs.py
  Input:  school_prompts/*.txt + scenarios.json
  Output: data/training/{school}_train.jsonl
  How:    Call Claude API with school prompt + scenario
  Time:   1-2 days to write, ~$20-50 in API costs

Script 2: train.py
  Input:  data/training/{school}_train.jsonl
  Output: adapters/{model}-{school}-lora/
  How:    Unsloth QLoRA (copy from CLAUDE.md, it's already written)
  Time:   1 day to set up, 1-2 hours to train per adapter

Script 3: app.py
  Input:  User's UX question + model + school selection
  Output: School-flavored analysis
  How:    Gradio + adapter hot-swap
  Time:   1 day
```

Total: 3-4 days to a working POC.
No extraction pipeline needed for Phase 1.

### Why no extraction for Phase 1?

The extracted book/web content feeds into either:
(A) System prompts — you can write these by hand faster
(B) Content-grounded pairs — synthetic scenario pairs work fine for POC

Extraction becomes valuable when you need:
  - 1000+ high-quality pairs (Phase 2)
  - RAG grounding with citations (Phase 3)
  - Community-contributed data at scale (Phase 3+)

### THEN Modularize for Phase 2+

Once POC proves distinctiveness, THEN build the modular pipeline:

```
Phase 2 architecture:

  Extraction Layer (use existing tools):
    - Unstructured.io or LlamaIndex for PDF/web/etc
    - youtube-transcript-api for transcripts
    - Semantic Scholar API for papers
    - Output: Universal chunk format (your idea — good)

  Preparation Layer (your value-add):
    - Scenario-based pair generation (Strategy A)
    - Content-grounded pair generation (Strategy B)
    - LLM-as-judge quality filtering
    - Output: JSONL training pairs

  Consumption Layer (plug-and-play):
    - FT pathway:   JSONL → Unsloth QLoRA → adapter
    - RAG pathway:   Chunks → embed → vector store
    - RAFT pathway:  Chunks + pairs → RAFT training
    - Eval pathway:  Pairs → distinctiveness test
```

---

## HAT 5: TREND CHECK — "What's the Industry Doing?"

### Trend 1: RAG is eating fine-tuning for most use cases

The consensus in 2025-2026:
  - Use RAG for knowledge injection
  - Use FT for style/behavior/reasoning changes
  - Use RAFT when you need both

Your project is about STYLE (design philosophy), not KNOWLEDGE.
That actually makes fine-tuning the RIGHT choice here.
The schools aren't about knowing different facts —
they're about REASONING DIFFERENTLY about the same facts.

Score: Your approach aligns with industry best practice.

### Trend 2: Modular, composable pipelines are the standard

Haystack, LlamaIndex, DSPy, Unstructured all emphasize:
  - Plug-and-play components
  - Universal data formats between stages
  - Swap models/stores without rewriting code

Your universal chunk format idea is EXACTLY right.
But: use existing tools as the components, don't rebuild them.

### Trend 3: Synthetic data is mainstream

Using strong LLMs to generate training data for smaller models
is now standard practice (distillation). Your approach is valid.

Key insight from latest research: quality filtering matters
MORE than volume. 300 excellent pairs beat 1000 mediocre ones.
Your LLM-as-judge pipeline is critical. Double down on it.

### Trend 4: RAFT is promising but niche

RAFT variants are multiplying (CRAFT, ALoFTRAG, GraphRAFT, RbFT)
but it's still an emerging technique, not a standard.
Good to have on the roadmap. Wrong to build for in Phase 1.

### Trend 5: No one is doing what DesignLens does

I searched for similar projects. The AI+UX space has:
  - UX Pilot AI (generates wireframes)
  - Galileo AI (generates screens)
  - Figma AI (design tool copilot)

None of them do "design philosophy reasoning through swappable lenses."
That's a genuine gap. The closest thing is asking ChatGPT
"analyze this like Nielsen" — which is just prompting.

Your value proposition (curated adapters + compare mode) is unique.
Don't dilute it by over-engineering the plumbing.

---

## SUMMARY: What to Change

| Your Idea | Verdict | Recommendation |
|-----------|---------|----------------|
| Modular tools | GOOD idea, WRONG phase | Build ugly POC first, modularize Phase 2 |
| Universal chunk format | EXCELLENT | Keep this as the contract between stages |
| Tool 1 (Extract) | DON'T BUILD | Use Unstructured.io / LlamaIndex |
| Tool 2 (Pair Gen) | BUILD THIS | This is your core differentiator |
| Tool 3 (FT) | USE UNSLOTH | Don't over-wrap it |
| Tool 4 (RAG) | DON'T BUILD | Use LlamaIndex + Chroma/FAISS |
| Tool 5 (RAFT) | DEFER to Phase 3 | Exciting but premature |
| Gemma 2B as base | OK for POC | Expect modest quality, that's fine |
| 500 synthetic pairs | OK for POC | Quality filter aggressively (keep top 300) |

### The One Thing to Do First

Before writing ANY code, run this test:
  1. Take 20 UX scenarios
  2. Send each to Claude with nielsen.txt system prompt
  3. Send each to Claude with norman.txt system prompt
  4. Have someone (or LLM judge) classify which school wrote which
  5. If accuracy > 80% → your school prompts work, proceed
  6. If accuracy < 60% → fix prompts before anything else

This test costs ~$2 and takes 30 minutes.
It validates the entire premise of the project.

---

## Sources

- RAFT paper: https://arxiv.org/abs/2403.10131
- RAFT explained (Microsoft): https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/raft-a-new-way-to-teach-llms-to-be-better-at-rag/4084674
- FT vs RAG decision framework: https://medium.com/@candemir13/fine-tuning-vs-rag-a-decision-framework-for-practitioners-7c26cba89768
- Meta's guide on when to fine-tune: https://ai.meta.com/blog/when-to-fine-tune-llms-vs-other-techniques/
- Unsloth FT guide: https://docs.unsloth.ai/get-started/fine-tuning-for-beginners/faq-+-is-fine-tuning-right-for-me
- Top RAG frameworks 2026: https://www.firecrawl.dev/blog/best-open-source-rag-frameworks
- AI UX tools landscape: https://www.figma.com/resource-library/ai-tools-for-ux-designers/
- Unstructured.io: https://unstructured.io/
