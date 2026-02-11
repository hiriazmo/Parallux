# Pivot Ideation: VLM + SLM Specialized for UX/Design

## The New Idea

Forget school-of-thought lenses for now.
What if you build a MODEL (or pair of models) that is simply
THE BEST at understanding and analyzing UX/Design?

Two components:
- VLM: Looks at screenshots/wireframes and gives visual UX feedback
- SLM: A small language model deeply specialized in UX domain knowledge

The school-of-thought lenses could become a FEATURE of this model later,
not the entire product.

---

## Why This Might Be Bigger

| DesignLens v1 (School Lenses) | DesignLens v2 (UX-Specialized VLM+SLM) |
|-------------------------------|----------------------------------------|
| Niche academic concept | Solves a daily practitioner problem |
| "Cool demo" energy | "I'd pay for this" energy |
| Text-only analysis | Can SEE your actual designs |
| Limited audience (who knows Nielsen vs Norman?) | Any designer, PM, or developer |
| Competes with: a good system prompt | Competes with: expensive UX consultants |

---

## What Exists Today (The Competition)

### Screenshot Analysis Tools (2026)

TOOL: Attention Insight
  DOES: AI heatmaps predicting where users look
  DOESN'T: Give actionable UX recommendations
  GAP: Sees WHERE attention goes, not WHY it's a problem

TOOL: UX Pilot (Figma plugin)
  DOES: Automated UX reviews, predictive heatmaps
  DOESN'T: Deep reasoning about WHY something is bad
  GAP: Surface-level feedback, not expert-level analysis

TOOL: Neurons Predict (formerly VisualEyes)
  DOES: Attention maps, clarity scores
  DOESN'T: Understand design patterns or UX principles
  GAP: Quantitative but not qualitative

TOOL: Emergent
  DOES: Analyzes broken UI, rewrites it in code
  DOESN'T: Explain the UX reasoning
  GAP: Fixes symptoms, doesn't diagnose root causes

TOOL: Figma AI
  DOES: Generate designs, write copy, compliance checks
  DOESN'T: Critique existing designs deeply
  GAP: Creation tool, not analysis tool

### The Gap No One Fills

None of these tools can:
  1. Look at a screenshot
  2. Identify UX problems with expert-level reasoning
  3. Explain WHY it's a problem (citing principles/heuristics)
  4. Prioritize by severity
  5. Suggest specific, actionable fixes
  6. Do this with the depth of a senior UX consultant

That's your opportunity.

---

## The VLM Side: "A Model That Can SEE Design Problems"

### What Would It Do?

INPUT: A screenshot (mobile app, website, dashboard, etc.)
OUTPUT: Expert UX analysis including:
  - Layout/hierarchy issues
  - Accessibility problems (contrast, target sizes, etc.)
  - Usability violations (unclear CTAs, confusing navigation)
  - Emotional/visual design feedback
  - Severity ratings
  - Specific fix recommendations

### Example

INPUT: [screenshot of a cluttered dashboard with tiny text,
        no visual hierarchy, 15 cards all the same size]

OUTPUT:
  "Critical Issues:
   1. NO VISUAL HIERARCHY (Severity: High)
      All 15 cards are identical size/weight. Users can't scan
      for what matters. The most-used 3 widgets (per your analytics)
      should be 2x larger and positioned top-left.

   2. TEXT SIZE BELOW ACCESSIBILITY MINIMUM (Severity: High)
      Body text appears to be ~10px. WCAG AA requires minimum
      16px for body text. The secondary labels are likely ~8px,
      making them unreadable for users with mild vision impairment.

   3. COLOR CONTRAST FAILURE (Severity: Medium)
      The light gray text (#999) on white (#FFF) background has
      a contrast ratio of ~2.8:1. WCAG AA requires 4.5:1 minimum.

   4. NO CLEAR ENTRY POINT (Severity: Medium)
      There's no primary CTA or clear first action. Users landing
      here will experience 'analysis paralysis' — 15 equal options
      means no obvious next step.

   Recommended Priority: Fix #1 and #2 first (highest impact)."

### Which Base VLM?

Best candidates for fine-tuning in 2025-2026:

MODEL: Qwen2.5-VL-7B / Qwen3-VL-8B
  WHY: Best open-source VLM for UI understanding
  PROOF: Already used for GUI agents, phone/computer use
  FT SUPPORT: LoRA/QLoRA supported, well-documented
  VRAM: ~24GB for LoRA fine-tuning at 1024px
  DATA NEEDED: 1,200-2,500 examples is the sweet spot (from real-world FT study)

MODEL: PaliGemma 2 (3B)
  WHY: Smallest viable VLM, good at localization
  PROOF: Excels at detecting/grounding specific elements
  FT SUPPORT: Quick to fine-tune with minimal data
  VRAM: ~8-12GB for LoRA
  LIMITATION: Weaker on long-form text generation

MODEL: LLaVA-OneVision (7B)
  WHY: Strong general-purpose VLM
  FT SUPPORT: Good community support
  LIMITATION: Less specialized for UI understanding

RECOMMENDATION FOR PHASE 1:
  Qwen2.5-VL-7B — best balance of UI understanding,
  fine-tuning support, and community documentation.

---

## The SLM Side: "A Small Model That Thinks Like a UX Expert"

### What Would It Do?

A text-only small model (2B-7B) that's deeply specialized
in UX/design knowledge. Think of it as:
  - Knows all of Nielsen's heuristics and when to apply them
  - Knows WCAG accessibility guidelines cold
  - Knows common design patterns and anti-patterns
  - Can reason about information architecture
  - Can suggest research methods for different problems
  - Speaks fluent UX vocabulary

### Why a Separate SLM?

The VLM handles: "What do I SEE in this screenshot?"
The SLM handles: "Given these observations, what's the UX diagnosis?"

Separation gives you:
  - VLM can be swapped/upgraded independently
  - SLM can work text-only too (describe a problem, get analysis)
  - Cheaper inference (SLM is small and fast)
  - The school-of-thought lenses become SLM adapters later

### OR: One VLM That Does Both

Simpler architecture: fine-tune Qwen2.5-VL-7B to both
SEE the screenshot AND reason about UX principles.

Trade-off:
  COMBINED: Simpler, one model, one inference call
  SEPARATE: More modular, can upgrade independently, SLM is cheaper for text-only

For Phase 1 POC: go COMBINED. One model. Prove it works.
Split later if needed.

---

## Training Data: What Would You Need?

This is the hard part. And your real moat.

### Dataset Type 1: Screenshot + UX Analysis Pairs

FORMAT:
  {
    "image": "screenshot_0042.png",
    "conversations": [
      {
        "role": "user",
        "content": "<image>\nAnalyze this UI for UX issues."
      },
      {
        "role": "assistant",
        "content": "1. VISUAL HIERARCHY: The page lacks... 2. ACCESSIBILITY: The contrast ratio..."
      }
    ]
  }

WHERE TO GET SCREENSHOTS:
  - RICO dataset (66K unique Android UI screenshots)
    https://interactionmining.org/rico
  - WebUI dataset (400K web page screenshots)
  - Figma Community files (publicly shared designs)
  - Your own collection of good vs bad UX examples
  - Screenshots from UX case studies (with analysis)

WHERE TO GET ANALYSES:
  - Use Claude/GPT-4V to analyze screenshots with a UX expert prompt
  - Annotate manually (Riaz's expertise — the moat)
  - Scrape UX teardown sites (growth.design, uxcam case studies)
  - Create from existing UX audit reports

ESTIMATED NEED: 1,500-3,000 pairs for a meaningful LoRA fine-tune

### Dataset Type 2: UX Knowledge Q&A (text-only)

FORMAT:
  {
    "instruction": "What is the difference between an affordance and a signifier?",
    "output": "An affordance is what an object allows you to do..."
  }

SOURCES:
  - UX textbooks (extracted and turned into Q&A)
  - NN/g articles (turned into Q&A pairs)
  - WCAG guidelines (turned into practical Q&A)
  - Design pattern libraries (turned into "when to use X" Q&A)
  - Common UX interview questions (already exist as datasets)

### Dataset Type 3: UX Problem Diagnosis

FORMAT:
  {
    "instruction": "Our checkout has 68% abandonment. Diagnose the UX issues.",
    "output": "Based on the symptoms, likely causes include..."
  }

This is basically what DesignLens v1 already planned.
It becomes ONE component of the training data, not the whole thing.

---

## How School-of-Thought Fits Into This

The schools don't disappear. They become a FEATURE:

BASIC MODE (default):
  "Analyze this screenshot for UX issues"
  → General expert-level UX analysis

LENS MODE (advanced feature):
  "Analyze this screenshot through Nielsen's lens"
  → Same VLM, but with a lens-specific system prompt or adapter
  → Emphasizes heuristic evaluation, severity ratings

COMPARE MODE:
  "Compare analyses from Nielsen, Norman, and Inclusive perspectives"
  → Run through multiple prompts/adapters, show side by side

The school lenses become a POWER USER feature of a broader
UX analysis tool, not the entire product.

---

## Architecture Options

### Option A: Single Fine-Tuned VLM (Simplest)

  Qwen2.5-VL-7B
    + LoRA fine-tuned on UX screenshot analysis data
    + Optional: school-specific system prompts at inference

  User uploads screenshot → model analyzes → returns feedback

  PRO: Simple, one model, one inference call
  CON: Need GPU for inference, 7B is heavy for free hosting

### Option B: VLM + SLM Pipeline

  Step 1: Qwen2.5-VL-7B describes what it sees
    "I see a login page with: email field, password field,
     submit button (blue, small), no forgot password link,
     no show/hide toggle, CAPTCHA below fold..."

  Step 2: UX-specialized SLM (Gemma 2B or Phi-3 mini) diagnoses
    "Based on these observations:
     1. Missing 'forgot password' violates error recovery...
     2. No show/hide toggle increases password errors...
     3. Small submit button may have touch target issues..."

  PRO: SLM is cheap and fast, can run on CPU
  PRO: VLM and SLM upgrade independently
  PRO: SLM works for text-only queries too
  CON: Two models, more complexity, potential information loss in handoff

### Option C: API-Powered VLM + Fine-Tuned SLM (Pragmatic)

  Step 1: Use Claude/GPT-4V API for vision (screenshot analysis)
    Already excellent at seeing UI issues, no fine-tuning needed

  Step 2: Fine-tuned SLM for deep UX reasoning
    Takes the visual description + produces expert analysis
    This is where school-of-thought adapters live

  PRO: Best vision quality immediately (no VLM fine-tuning needed)
  PRO: SLM is your differentiator (domain expertise)
  PRO: Cheapest to build POC
  CON: Depends on API for vision (cost per call)
  CON: Less "open source" since vision depends on proprietary API

### Option D: The "2026 is the year of fine-tuned small models" Play

  Fine-tune Qwen3-VL-8B as a complete UX analysis VLM.
  Open-source the adapter + training data on HuggingFace.
  Become the "go-to UX model" the way CodeLlama is for code.

  PRO: Maximum open-source impact and community adoption
  PRO: Single model, clean story
  CON: Needs significant training data (2K+ annotated screenshots)
  CON: Needs GPU for inference

---

## Competitive Positioning

WHERE THIS FITS IN THE MARKET:

  Existing AI UX tools:
    - Heatmaps (Attention Insight, Neurons Predict)
    - Generation (Figma AI, Galileo, Uizard)
    - Code from screenshots (Emergent, v0)
    - Surface-level review (UX Pilot)

  What DOESN'T exist:
    - An open-source VLM fine-tuned specifically for UX analysis
    - A model that gives EXPERT-LEVEL reasoning, not just "this has low contrast"
    - A model that understands design PRINCIPLES, not just pixels
    - Something the community can download, run locally, and improve

  Your positioning:
    "The CodeLlama of UX — an open-source model that actually
     understands design, not just sees pixels."

---

## What I'd Build (If I Were You)

### Phase 1 (Weeks 1-3): Validate with existing VLMs

  DON'T fine-tune yet. Test the gap:
  1. Collect 50 UI screenshots (mix of good and bad UX)
  2. Send each to GPT-4V, Claude, Gemini with a UX expert prompt
  3. Also send to Qwen2.5-VL-7B (base, no fine-tuning)
  4. Compare: How good is BASE model vs API models at UX analysis?
  5. The GAP between base and API = what fine-tuning can improve

  This tells you:
  - If base VLMs already good enough → just build the UX prompt toolkit
  - If base VLMs are weak on UX → fine-tuning has clear value
  - What SPECIFIC UX skills are missing (accessibility, hierarchy, etc.)

### Phase 2 (Weeks 4-8): Build the training dataset

  This is the REAL work and your real moat:
  1. Collect 2,000 UI screenshots across platforms and quality levels
  2. Generate expert UX analyses using Claude/GPT-4V + UX expert prompt
  3. Have Riaz review and correct 200-300 of them (gold standard)
  4. Use corrected examples to improve the generation prompt
  5. Quality filter with LLM-as-judge
  6. Result: 1,500+ screenshot-analysis pairs

### Phase 3 (Weeks 9-12): Fine-tune and release

  1. LoRA fine-tune Qwen2.5-VL-7B (or Qwen3-VL) on the dataset
  2. Evaluate: fine-tuned vs base vs API models
  3. Add school-of-thought adapters as bonus feature
  4. Build Gradio demo
  5. Release on HuggingFace: model + dataset + benchmark

---

## The Killer Question

Is the value in the MODEL or the DATASET?

If you build a high-quality dataset of 2,000 UI screenshots with
expert UX analyses, that dataset is valuable REGARDLESS of which
model you fine-tune. It becomes:
  - Training data for any VLM
  - Benchmark for evaluating UX capabilities
  - Teaching resource for UX students
  - The first open UX analysis dataset on HuggingFace

The dataset might be more valuable than the model.
Models get outdated. Curated datasets compound in value.

---

## Sources

- CANVAS benchmark (VLM for UI design):
  https://arxiv.org/abs/2511.20737
  https://canvas.kixlab.org/
- Qwen2.5-VL fine-tuning tutorial:
  https://skywork.ai/blog/llm/fine-tune-qwen2-5-vl-32b-in-3-days-complete-hands-on-tutorial/
- Qwen3-VL on Unsloth:
  https://docs.unsloth.ai/models/qwen3-vl-how-to-run-and-fine-tune
- VLMs 2025 overview (HuggingFace):
  https://huggingface.co/blog/vlms-2025
- Top VLMs 2026:
  https://dextralabs.com/blog/top-10-vision-language-models/
- 2026 is the year of fine-tuned small models:
  https://seldo.com/posts/2026-is-the-year-of-fine-tuned-small-models
- Why not fine-tune (counterargument):
  https://eclipsesource.com/blogs/2025/05/08/why-not-fine-tune-llms/
- Fine-tuning SLMs for domain-specific AI (paper):
  https://arxiv.org/html/2503.01933v1
- RICO dataset (66K Android UI screenshots):
  https://interactionmining.org/rico
- AI tools for UX designers (Figma):
  https://www.figma.com/resource-library/ai-tools-for-ux-designers/
- Top AI tools for UI design 2026:
  https://emergent.sh/learn/best-ai-tools-for-ui-design
- UX Pilot:
  https://uxpilot.ai/
- Attention Insight:
  https://www.attentioninsight.com/
