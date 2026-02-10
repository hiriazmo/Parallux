# CLAUDE.md — DesignLens: Multi-Model Design Philosophy Engine

> **Project Codename:** DesignLens
> **Author:** Riaz
> **Status:** Phase 1 — POC
> **Last Updated:** 2025-02-10

---

## 1. PROJECT VISION

DesignLens is an open-source, multi-model UX intelligence platform that analyzes design problems through **swappable design philosophy lenses**. Each "lens" is a QLoRA adapter fine-tuned on a specific design school of thought. Users select a base model + design school from the UI, and the system responds with analysis grounded in that philosophy.

**End Goal:** The go-to open-source UX AI toolkit on HuggingFace — a collection of models, adapters, datasets, and evaluation benchmarks that the design community can use and contribute to.

---

## 2. HIGH-LEVEL ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────┐
│                      FRONTEND (Gradio / Streamlit)          │
│                                                             │
│   [Model Selector]  ×  [Design School Selector]             │
│   [Compare All Mode]   [Blend Mode]   [Input Area]         │
└──────────────────────────┬──────────────────────────────────┘
                           │ API Call
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    INFERENCE ENGINE                          │
│                                                             │
│   ┌─────────────┐   ┌──────────────┐   ┌───────────────┐  │
│   │ Model Loader │   │ Adapter Swap │   │ Response Gen  │  │
│   │ (cached)     │──▶│ (hot-swap)   │──▶│ (generate)    │  │
│   └─────────────┘   └──────────────┘   └───────────────┘  │
│                                                             │
│   Adapter Registry: model × school → adapter_path           │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    ADAPTER STORE (HuggingFace Hub)           │
│                                                             │
│   riaz/designlens-gemma2b-nielsen-lora                      │
│   riaz/designlens-gemma2b-norman-lora                       │
│   riaz/designlens-llama3-8b-nielsen-lora                    │
│   riaz/designlens-llama3-8b-norman-lora                     │
│   ...                                                       │
└─────────────────────────────────────────────────────────────┘
```

### Optional RAG Layer (Phase 3+)

```
User Query
    │
    ├──▶ Adapter (school-specific reasoning)
    │
    └──▶ RAG Retrieval (school-specific vector store)
              │
              ▼
         Grounded citations from source books/articles
```

---

## 3. DESIGN SCHOOLS (ADAPTERS)

### Phase 1 (POC) — 2 Schools

| ID | School | Core Thinker | Key Framework |
|----|--------|-------------|---------------|
| `nielsen` | Usability / Heuristic | Jakob Nielsen | 10 Usability Heuristics, task efficiency, error prevention |
| `norman` | Emotional / Cognitive | Don Norman | Visceral-Behavioral-Reflective, affordances, conceptual models |

### Phase 2 — Expand to 6 Schools

| ID | School | Core Thinker | Key Framework |
|----|--------|-------------|---------------|
| `rams` | Minimalism / Functionalism | Dieter Rams | 10 Principles of Good Design, less but better |
| `inclusive` | Inclusive / Accessibility | Kat Holmes | Solve for one, extend to many; WCAG; mismatch model |
| `service` | Service Design | Stickdorn | Blueprints, touchpoints, frontstage/backstage |
| `behavioral` | Behavioral / Nudge | Thaler, Eyal | Choice architecture, nudges, hook model, dark patterns |

### Phase 3+ — Community Contributed

| ID | School | Core Thinker | Key Framework |
|----|--------|-------------|---------------|
| `atomic` | Atomic Design | Brad Frost | Atoms → Molecules → Organisms → Templates → Pages |
| `jtbd` | Jobs To Be Done | Christensen | Functional/social/emotional jobs, hiring/firing metaphor |
| `lean` | Lean UX | Jeff Gothelf | Build-Measure-Learn, hypothesis-driven design |
| `speculative` | Speculative / Critical | Dunne & Raby | What if scenarios, design as provocation |
| `custom` | User-defined | Any | Upload principles → auto-generate adapter |

---

## 4. PHASED IMPLEMENTATION PLAN

### ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PHASE 1: POC (Weeks 1-3)

**Goal:** Prove the concept works — one base model, two schools, distinct outputs.

**Deliverables:**
- [ ] Data pipeline for 2 schools (nielsen + norman)
- [ ] 500-1000 training pairs per school
- [ ] QLoRA fine-tuned adapters on Gemma 2B via Unsloth
- [ ] Gradio demo with model + school selector
- [ ] Deployed on HuggingFace Spaces
- [ ] Distinctiveness evaluation (do schools actually differ?)

**Stack:**
- Base Model: `google/gemma-2b`
- Training: Unsloth + QLoRA
- UI: Gradio
- Hosting: HuggingFace Spaces (free tier)
- Data Gen: Claude API / GPT-4 for synthetic pairs

**Directory Structure:**
```
designlens/
├── CLAUDE.md                    # This file — project context
├── README.md                    # Public-facing docs
├── requirements.txt
├── pyproject.toml
│
├── data/
│   ├── raw/                     # Extracted raw text
│   │   ├── books/
│   │   ├── articles/
│   │   ├── transcripts/
│   │   └── case_studies/
│   │
│   ├── processed/               # Cleaned, chunked text
│   │   ├── nielsen/
│   │   └── norman/
│   │
│   ├── training/                # Final instruction-response pairs
│   │   ├── nielsen_train.jsonl
│   │   ├── nielsen_val.jsonl
│   │   ├── norman_train.jsonl
│   │   └── norman_val.jsonl
│   │
│   ├── scenarios/               # Reusable UX scenarios
│   │   └── scenarios.json
│   │
│   └── evaluation/              # Benchmark test sets
│       └── distinctiveness_test.jsonl
│
├── extraction/                  # Data extraction pipelines
│   ├── pdf_extractor.py         # Marker / Docling
│   ├── web_scraper.py           # Scrapy / BeautifulSoup
│   ├── video_transcriber.py     # Whisper
│   ├── paper_fetcher.py         # Semantic Scholar API
│   └── case_study_extractor.py
│
├── generation/                  # Synthetic data generation
│   ├── school_prompts/          # System prompts per school
│   │   ├── nielsen.txt
│   │   └── norman.txt
│   ├── generate_pairs.py        # LLM-powered pair generation
│   ├── generate_scenarios.py    # UX scenario generator
│   ├── quality_filter.py        # LLM-as-judge scoring
│   └── distinctiveness_test.py  # Cross-school differentiation check
│
├── training/                    # Fine-tuning scripts
│   ├── train_unsloth.py         # Unsloth QLoRA training
│   ├── train_jax_gemma.py       # JAX/Keras for Gemma on TPU
│   ├── train_hf_peft.py         # Standard HF PEFT fallback
│   ├── merge_adapter.py         # Merge LoRA into base (optional)
│   └── configs/
│       ├── gemma_2b.yaml
│       ├── llama3_8b.yaml
│       └── mistral_7b.yaml
│
├── inference/                   # Inference engine
│   ├── adapter_registry.py      # Model × School → adapter path
│   ├── model_loader.py          # Cached model loading
│   ├── adapter_swapper.py       # Hot-swap logic
│   ├── compare_mode.py          # Run all schools, return comparison
│   └── blend_mode.py            # Weighted adapter merging
│
├── app/                         # Frontend
│   ├── gradio_app.py            # Phase 1: Gradio UI
│   └── streamlit_app.py         # Phase 3+: Production UI
│
├── evaluation/                  # Eval & benchmarks
│   ├── eval_distinctiveness.py  # Are schools actually different?
│   ├── eval_quality.py          # Response quality scoring
│   ├── eval_faithfulness.py     # Does it follow the school's principles?
│   └── eval_report.py           # Generate eval report
│
└── notebooks/                   # Experimentation
    ├── 01_data_exploration.ipynb
    ├── 02_training_poc.ipynb
    ├── 03_eval_analysis.ipynb
    └── 04_jax_gemma_training.ipynb
```

### ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PHASE 2: Multi-Model + Expansion (Weeks 4-8)

**Goal:** Add more base models, more schools, compare mode.

**Deliverables:**
- [ ] Add Llama 3 8B and Mistral 7B as base models
- [ ] Expand to 5-6 design schools
- [ ] "Compare All" mode in UI
- [ ] JAX training pipeline for Gemma on TPU
- [ ] Comprehensive evaluation benchmarks
- [ ] Dataset published on HuggingFace

**New Stack Additions:**
- JAX + Keras 3 (for Gemma TPU training)
- Multiple base models via Unsloth

### ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PHASE 3: Open Source Release (Weeks 9-12)

**Goal:** Public release as HuggingFace Collection.

**Deliverables:**
- [ ] HuggingFace Collection with all adapters
- [ ] Model cards for every adapter
- [ ] Published training datasets
- [ ] Evaluation benchmark suite
- [ ] Contributing guide (how to add a new school)
- [ ] Blog post / demo video
- [ ] RAG layer for citation grounding (optional)

**HuggingFace Collection Structure:**
```
riaz/designlens/
├── designlens-gemma2b-nielsen-lora
├── designlens-gemma2b-norman-lora
├── designlens-gemma2b-rams-lora
├── designlens-llama3-nielsen-lora
├── designlens-llama3-norman-lora
├── designlens-llama3-rams-lora
├── designlens-mistral-nielsen-lora
├── designlens-mistral-norman-lora
├── designlens-training-datasets
├── designlens-eval-benchmark
└── designlens-spaces-demo
```

### ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

### PHASE 4: Production Product (Month 4+)

**Goal:** Turn into a real product.

**Deliverables:**
- [ ] Next.js / Streamlit production frontend
- [ ] Adapter blending (weighted merge of schools)
- [ ] "Create Your Own School" — upload principles → auto-train
- [ ] API access for tool integrations
- [ ] Figma plugin (analyze designs in context)
- [ ] Monetization: freemium model or sponsored adapters

---

## 5. DATA EXTRACTION PIPELINE — DETAILED

### 5.1 Source Inventory

```yaml
# data/sources.yaml

nielsen:
  books:
    - title: "Usability Engineering"
      author: "Jakob Nielsen"
      format: pdf
      priority: high
    - title: "Designing Web Usability"
      author: "Jakob Nielsen"
      format: pdf
      priority: high
  web:
    - url: "https://www.nngroup.com/articles/"
      type: article_archive
      estimated_articles: 2000+
      priority: high
    - url: "https://www.nngroup.com/videos/"
      type: video_archive
      priority: medium
  papers:
    - search_query: "usability heuristics evaluation"
      source: semantic_scholar
      priority: medium

norman:
  books:
    - title: "The Design of Everyday Things"
      author: "Don Norman"
      format: pdf
      priority: high
    - title: "Emotional Design"
      author: "Don Norman"
      format: pdf
      priority: high
    - title: "Living with Complexity"
      author: "Don Norman"
      format: pdf
      priority: medium
  web:
    - url: "https://jnd.org/articles/"
      type: blog
      priority: high
  videos:
    - search_query: "Don Norman design talk"
      source: youtube
      priority: medium
  papers:
    - search_query: "emotional design user experience affordance"
      source: semantic_scholar
      priority: medium
```

### 5.2 Extraction Methods

#### PDF / Book Extraction

```python
# extraction/pdf_extractor.py

"""
PDF Extraction Pipeline
-----------------------
Tools (in priority order):
1. Marker  — Best general-purpose PDF → Markdown
   pip install marker-pdf
   
2. Docling (IBM) — Best for structured/academic documents
   pip install docling
   
3. PyMuPDF + pdfplumber — Fallback for simple PDFs
   pip install pymupdf pdfplumber

4. Nougat (Meta) — Academic papers with equations
   pip install nougat-ocr

Output: Clean Markdown per chapter/section
"""

from marker.convert import convert_single_pdf
from pathlib import Path
import json

def extract_pdf(pdf_path: str, output_dir: str, school: str):
    """Extract PDF to structured markdown chunks."""
    
    # Step 1: Convert PDF → Markdown
    full_text, metadata = convert_single_pdf(pdf_path)
    
    # Step 2: Chunk by chapters/sections
    chunks = chunk_by_headers(full_text)
    
    # Step 3: Save with metadata
    for i, chunk in enumerate(chunks):
        output = {
            "source": pdf_path,
            "school": school,
            "chunk_id": i,
            "content": chunk["text"],
            "section": chunk["header"],
            "type": "book"
        }
        save_path = Path(output_dir) / school / f"book_chunk_{i:04d}.json"
        save_path.parent.mkdir(parents=True, exist_ok=True)
        save_path.write_text(json.dumps(output, indent=2))

def chunk_by_headers(markdown_text: str, max_chunk_size: int = 2000):
    """Split markdown by headers, respecting max chunk size."""
    # Split on ## or # headers
    # Keep chunks under max_chunk_size tokens
    # Preserve context by including header hierarchy
    pass
```

#### Web Scraping

```python
# extraction/web_scraper.py

"""
Web Scraping Pipeline
---------------------
Targets:
- NN/g articles (nielsen school)
- jnd.org (norman school)
- Smashing Magazine, A List Apart (multiple schools)
- Laws of UX (multiple schools)
- Growth.design case studies

Tools:
- requests + BeautifulSoup (simple sites)
- Playwright (JS-rendered sites)
- trafilatura (article extraction)
- markdownify (HTML → Markdown)

Rate limiting: Respect robots.txt, 2-3 sec delay between requests.
"""

import requests
from bs4 import BeautifulSoup
from trafilatura import fetch_url, extract
import time
import json
from pathlib import Path

def scrape_nng_articles(output_dir: str, max_articles: int = 200):
    """Scrape NN/g articles for Nielsen school training data."""
    
    base_url = "https://www.nngroup.com/articles/"
    articles = []
    
    # Step 1: Get article URLs from listing pages
    article_urls = get_article_urls(base_url, max_pages=20)
    
    # Step 2: Extract each article
    for url in article_urls[:max_articles]:
        downloaded = fetch_url(url)
        content = extract(downloaded, include_comments=False)
        
        if content:
            articles.append({
                "url": url,
                "content": content,
                "school": "nielsen",
                "type": "web_article",
                "source": "nngroup"
            })
        
        time.sleep(2)  # Respect rate limits
    
    # Step 3: Save
    output_path = Path(output_dir) / "nielsen" / "nng_articles.jsonl"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    with open(output_path, "w") as f:
        for article in articles:
            f.write(json.dumps(article) + "\n")
    
    return len(articles)
```

#### Video Transcription

```python
# extraction/video_transcriber.py

"""
Video Transcription Pipeline
-----------------------------
Tools:
- faster-whisper (recommended — 4x faster than OpenAI Whisper)
  pip install faster-whisper
  
- yt-dlp for YouTube downloads
  pip install yt-dlp

Target: Conference talks, design lectures, interviews
"""

from faster_whisper import WhisperModel
import subprocess
import json

def transcribe_youtube(video_url: str, school: str, output_dir: str):
    """Download and transcribe a YouTube video."""
    
    # Step 1: Download audio
    audio_path = f"/tmp/audio_{hash(video_url)}.mp3"
    subprocess.run([
        "yt-dlp", "-x", "--audio-format", "mp3",
        "-o", audio_path, video_url
    ])
    
    # Step 2: Transcribe
    model = WhisperModel("large-v3", device="cuda", compute_type="float16")
    segments, info = model.transcribe(audio_path)
    
    transcript = " ".join([seg.text for seg in segments])
    
    # Step 3: Save
    output = {
        "source_url": video_url,
        "school": school,
        "type": "video_transcript",
        "content": transcript,
        "language": info.language,
        "duration_seconds": info.duration
    }
    
    return output
```

#### Academic Paper Fetcher

```python
# extraction/paper_fetcher.py

"""
Academic Paper Pipeline
-----------------------
Tools:
- Semantic Scholar API (free, no auth needed for basic)
- Nougat / Marker for PDF extraction

Best sources:
- CHI conference papers (HCI/UX gold standard)
- DIS (Designing Interactive Systems)
- NordiCHI, CSCW
"""

import requests
import time

SEMANTIC_SCHOLAR_API = "https://api.semanticscholar.org/graph/v1"

def search_papers(query: str, limit: int = 50):
    """Search Semantic Scholar for relevant papers."""
    
    url = f"{SEMANTIC_SCHOLAR_API}/paper/search"
    params = {
        "query": query,
        "limit": limit,
        "fields": "title,abstract,year,authors,url,openAccessPdf"
    }
    
    response = requests.get(url, params=params)
    papers = response.json().get("data", [])
    
    # Filter for open access PDFs
    accessible = [p for p in papers if p.get("openAccessPdf")]
    
    return accessible

def download_and_extract(paper: dict, school: str):
    """Download paper PDF and extract text."""
    pdf_url = paper["openAccessPdf"]["url"]
    # Download PDF → Extract with Marker/Nougat
    # Return structured content
    pass
```

---

## 6. SYNTHETIC DATA GENERATION

### 6.1 School-Specific System Prompts

```
# generation/school_prompts/nielsen.txt

You are a senior UX analyst who strictly follows Jakob Nielsen's usability 
principles and the Nielsen Norman Group methodology. 

YOUR CORE FRAMEWORK:
- 10 Usability Heuristics as your primary evaluation lens
- Task-based thinking: efficiency, learnability, memorability, errors, satisfaction
- Data-driven: always reference metrics, benchmarks, task completion rates
- Severity ratings for usability issues (cosmetic → catastrophic)
- Think aloud protocol, heuristic evaluation, cognitive walkthrough

YOUR REASONING PATTERN:
1. Identify which heuristic(s) are violated
2. Assess severity (0-4 scale)
3. Provide evidence-based recommendation
4. Reference comparable benchmarks or studies
5. Prioritize by impact on task completion

YOUR VOCABULARY:
- Heuristics, learnability, efficiency, memorability, error rate
- Task completion, time-on-task, satisfaction score
- Cognitive load, recognition over recall, error prevention
- Consistency, standards, flexibility, minimalist design

YOU NEVER:
- Discuss emotions or aesthetics as primary concerns
- Recommend changes without usability justification
- Ignore measurable outcomes
```

```
# generation/school_prompts/norman.txt

You are a senior UX analyst who strictly follows Don Norman's design 
philosophy from "The Design of Everyday Things" and "Emotional Design."

YOUR CORE FRAMEWORK:
- Three levels of processing: Visceral (immediate), Behavioral (use), 
  Reflective (memory/meaning)
- Seven Stages of Action model
- Gulf of Execution and Gulf of Evaluation
- Affordances, Signifiers, Constraints, Mappings, Feedback, Conceptual Models

YOUR REASONING PATTERN:
1. Identify the emotional experience at each level (visceral/behavioral/reflective)
2. Trace the breakdown to a specific design concept (affordance, signifier, etc.)
3. Analyze the conceptual model mismatch
4. Propose solutions that address both function AND emotion
5. Consider how the redesign affects the user's self-image and memory

YOUR VOCABULARY:
- Affordance, signifier, constraint, mapping, feedback
- Conceptual model, system image, mental model
- Visceral, behavioral, reflective
- Gulf of execution, gulf of evaluation
- Knowledge in the head vs knowledge in the world
- Emotional design, pleasure, meaning, self-identity

YOU ALWAYS:
- Start with how the design makes users FEEL
- Consider the full emotional arc of the experience
- Balance usability with emotional impact
- Think about what the design communicates about the user's identity
```

### 6.2 Scenario Generation

```python
# generation/generate_scenarios.py

"""
Generate diverse UX scenarios to use as inputs for training pair generation.
Goal: 200+ unique scenarios covering different domains, platforms, and problem types.
"""

UX_SCENARIOS = {
    "ecommerce": [
        "E-commerce checkout page has a 68% cart abandonment rate. Users add items but don't complete purchase.",
        "Product search returns relevant items but conversion from search to PDP is only 15%.",
        "Mobile product listing page has 3x higher bounce rate than desktop.",
        "Size selector on clothing PDP causes 40% of customer support tickets.",
        "Wishlist feature has high add rate but near-zero conversion to purchase.",
    ],
    "saas": [
        "SaaS onboarding flow has 12 steps and only 23% completion rate.",
        "Dashboard has 15 widgets but analytics show users only interact with 3.",
        "Feature adoption for newly launched collaboration tool is under 5%.",
        "Settings page receives highest 'rage click' count in the entire app.",
        "Trial-to-paid conversion drops at the payment information step.",
    ],
    "healthcare": [
        "Patient portal appointment booking takes average 4.5 minutes to complete.",
        "Medication refill flow has a 30% error rate in dosage selection.",
        "Elderly users abandon telehealth setup at camera/microphone permission step.",
        "Lab results page causes highest support call volume.",
        "Insurance verification form has 60% incomplete submission rate.",
    ],
    "attractions": [
        "Theme park ticket booking flow has 5 decision points causing analysis paralysis.",
        "Mobile app wayfinding feature shows 2-star rating in app stores.",
        "Annual pass renewal rate dropped 15% after website redesign.",
        "Add-on upsell during ticket purchase has 2% conversion vs industry 12%.",
        "Group booking flow doesn't support mixed ticket types.",
    ],
    "fintech": [
        "Banking app money transfer flow has higher error rate on mobile than desktop.",
        "Investment portfolio page shows too much data — users miss key alerts.",
        "KYC onboarding document upload fails 35% of the time on older phones.",
        "Budget tracking feature has high initial setup but low daily return rate.",
        "Credit score page has highest screenshot rate — users share it externally.",
    ],
    "general": [
        "Navigation menu restructure needed — information architecture is flat with 40+ items.",
        "Design system components aren't being adopted by development teams.",
        "Accessibility audit found 47 WCAG violations across the main user flow.",
        "A/B test shows controversial redesign increases conversion but decreases satisfaction.",
        "Multi-language support causing layout breaks in RTL languages.",
    ]
}

QUESTION_TYPES = [
    "Analyze this UX problem and provide your assessment.",
    "How would you redesign this experience?",
    "What research methods would you use to investigate this issue?",
    "Critique this current design and prioritize improvements.",
    "What metrics would you track to measure improvement?",
    "Create a brief design recommendation document for stakeholders.",
    "Identify the root cause of this user behavior.",
    "What quick wins and long-term fixes would you recommend?",
]
```

### 6.3 Pair Generation Pipeline

```python
# generation/generate_pairs.py

"""
Generate instruction-response pairs using LLM with school-specific prompts.

For each (scenario × question_type × school), generate one training pair.
This creates contrastive data — same question, different school = different answer.

Usage:
    python generate_pairs.py --school nielsen --num_pairs 500 --model claude
    python generate_pairs.py --school norman --num_pairs 500 --model gpt4
"""

import anthropic
import json
from pathlib import Path

def load_school_prompt(school: str) -> str:
    return Path(f"generation/school_prompts/{school}.txt").read_text()

def generate_pair(
    scenario: str,
    question_type: str,
    school: str,
    client: anthropic.Anthropic
) -> dict:
    """Generate one training pair."""
    
    system_prompt = load_school_prompt(school)
    
    user_message = f"""{question_type}

Context: {scenario}

Provide a thorough analysis strictly following your design philosophy. 
Be specific, actionable, and demonstrate your school of thought's 
unique reasoning pattern."""
    
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1500,
        system=system_prompt,
        messages=[{"role": "user", "content": user_message}]
    )
    
    return {
        "instruction": question_type,
        "input": scenario,
        "output": response.content[0].text,
        "school": school,
        "metadata": {
            "generation_model": "claude-sonnet-4-20250514",
            "scenario_domain": detect_domain(scenario),
            "question_type": question_type
        }
    }

def generate_dataset(school: str, num_pairs: int = 500):
    """Generate full training dataset for one school."""
    
    client = anthropic.Anthropic()
    pairs = []
    
    all_scenarios = flatten_scenarios(UX_SCENARIOS)
    
    for i in range(num_pairs):
        scenario = all_scenarios[i % len(all_scenarios)]
        question = QUESTION_TYPES[i % len(QUESTION_TYPES)]
        
        pair = generate_pair(scenario, question, school, client)
        pairs.append(pair)
        
        if i % 50 == 0:
            print(f"Generated {i}/{num_pairs} pairs for {school}")
    
    # Save
    output_path = Path(f"data/training/{school}_train.jsonl")
    output_path.parent.mkdir(parents=True, exist_ok=True)
    with open(output_path, "w") as f:
        for pair in pairs:
            f.write(json.dumps(pair) + "\n")
    
    return len(pairs)
```

### 6.4 Quality Filtering

```python
# generation/quality_filter.py

"""
LLM-as-Judge quality filtering pipeline.

Scores each training pair on:
1. School Faithfulness (1-5): Does it follow the school's principles?
2. Distinctiveness (1-5): Would you identify this as School X vs others?
3. Quality (1-5): Is the analysis useful, specific, actionable?
4. Consistency (1-5): Does it use appropriate vocabulary/frameworks?

Discard pairs with average score < 3.5
"""

JUDGE_PROMPT = """You are evaluating a UX analysis response for training data quality.

The response should reflect the {school} design philosophy.

SCHOOL DESCRIPTION:
{school_description}

SCENARIO: {scenario}
RESPONSE: {response}

Score each dimension 1-5:

1. SCHOOL_FAITHFULNESS: Does this response clearly follow {school} principles?
   1=Generic/wrong school, 5=Perfectly aligned with school

2. DISTINCTIVENESS: Could you identify which school wrote this without being told?
   1=Could be any school, 5=Unmistakably this school

3. QUALITY: Is the analysis useful, specific, and actionable?
   1=Vague/useless, 5=Expert-level analysis

4. CONSISTENCY: Does it use appropriate vocabulary and frameworks?
   1=Wrong terminology, 5=Perfect vocabulary alignment

Respond as JSON:
{{"school_faithfulness": N, "distinctiveness": N, "quality": N, "consistency": N, "reasoning": "brief explanation"}}
"""
```

---

## 7. TRAINING PIPELINE

### 7.1 Unsloth (Primary — POC + Production)

```python
# training/train_unsloth.py

"""
QLoRA fine-tuning with Unsloth.
2x faster, 60% less memory than standard HF training.

Requirements:
    pip install unsloth
    pip install trl transformers datasets

Usage:
    python train_unsloth.py --school nielsen --base_model gemma-2b
    python train_unsloth.py --school norman --base_model gemma-2b
"""

from unsloth import FastLanguageModel
from trl import SFTTrainer
from transformers import TrainingArguments
from datasets import load_dataset

# ─── Config ───────────────────────────────────
MODEL_CONFIGS = {
    "gemma-2b": {
        "model_name": "unsloth/gemma-2b-bnb-4bit",
        "max_seq_length": 2048,
        "lora_r": 16,
        "lora_alpha": 32,
        "lora_target_modules": [
            "q_proj", "k_proj", "v_proj", "o_proj",
            "gate_proj", "up_proj", "down_proj"
        ],
    },
    "llama3-8b": {
        "model_name": "unsloth/Meta-Llama-3-8B-bnb-4bit",
        "max_seq_length": 4096,
        "lora_r": 16,
        "lora_alpha": 32,
        "lora_target_modules": [
            "q_proj", "k_proj", "v_proj", "o_proj",
            "gate_proj", "up_proj", "down_proj"
        ],
    },
    "mistral-7b": {
        "model_name": "unsloth/mistral-7b-bnb-4bit",
        "max_seq_length": 4096,
        "lora_r": 16,
        "lora_alpha": 32,
        "lora_target_modules": [
            "q_proj", "k_proj", "v_proj", "o_proj",
            "gate_proj", "up_proj", "down_proj"
        ],
    }
}

TRAINING_TEMPLATE = """### Instruction:
{instruction}

### Input:
{input}

### Response:
{output}"""

def train(school: str, base_model: str = "gemma-2b"):
    config = MODEL_CONFIGS[base_model]
    
    # Step 1: Load model in 4-bit
    model, tokenizer = FastLanguageModel.from_pretrained(
        model_name=config["model_name"],
        max_seq_length=config["max_seq_length"],
        load_in_4bit=True,
    )
    
    # Step 2: Add LoRA adapters
    model = FastLanguageModel.get_peft_model(
        model,
        r=config["lora_r"],
        lora_alpha=config["lora_alpha"],
        target_modules=config["lora_target_modules"],
        lora_dropout=0.05,
        bias="none",
        use_gradient_checkpointing=True,
    )
    
    # Step 3: Load training data
    dataset = load_dataset(
        "json",
        data_files=f"data/training/{school}_train.jsonl",
        split="train"
    )
    
    def format_prompt(example):
        return TRAINING_TEMPLATE.format(**example)
    
    # Step 4: Train
    trainer = SFTTrainer(
        model=model,
        tokenizer=tokenizer,
        train_dataset=dataset,
        formatting_func=format_prompt,
        max_seq_length=config["max_seq_length"],
        args=TrainingArguments(
            output_dir=f"outputs/{base_model}-{school}",
            num_train_epochs=3,
            per_device_train_batch_size=4,
            gradient_accumulation_steps=4,
            learning_rate=2e-4,
            weight_decay=0.01,
            warmup_steps=50,
            logging_steps=10,
            save_strategy="epoch",
            fp16=True,
            optim="adamw_8bit",
        ),
    )
    
    trainer.train()
    
    # Step 5: Save adapter
    model.save_pretrained(f"adapters/{base_model}-{school}-lora")
    tokenizer.save_pretrained(f"adapters/{base_model}-{school}-lora")
    
    # Step 6: Push to HuggingFace Hub
    # model.push_to_hub(f"riaz/designlens-{base_model}-{school}-lora")
    
    print(f"✅ Training complete: {base_model} × {school}")
```

### 7.2 JAX / Keras (For Gemma on TPU)

```python
# training/train_jax_gemma.py

"""
JAX-based fine-tuning for Gemma models on TPU.
Use this when training on Google Colab/Kaggle with free TPU.

Requirements:
    pip install keras keras-nlp jax jaxlib

Advantage: Gemma is natively JAX — 2-3x faster on TPU vs PyTorch on GPU.
"""

import keras
import keras_nlp

def train_gemma_jax(school: str):
    # Load Gemma with LoRA
    gemma_lm = keras_nlp.models.GemmaCausalLM.from_preset("gemma_2b_en")
    
    # Enable LoRA
    gemma_lm.backbone.enable_lora(rank=16)
    
    # Compile with JAX
    gemma_lm.compile(
        loss=keras.losses.SparseCategoricalCrossentropy(from_logits=True),
        optimizer=keras.optimizers.Adam(learning_rate=2e-4),
        weighted_metrics=[keras.metrics.SparseCategoricalAccuracy()],
    )
    
    # Load and format data
    # ... (similar to Unsloth pipeline)
    
    # Train
    gemma_lm.fit(dataset, epochs=3, batch_size=4)
    
    # Save adapter weights
    gemma_lm.save_weights(f"adapters/gemma-2b-{school}-jax-lora")
```

---

## 8. INFERENCE ENGINE

### 8.1 Adapter Registry

```python
# inference/adapter_registry.py

"""
Central registry mapping (model, school) → adapter path.
Supports both local paths and HuggingFace Hub references.
"""

ADAPTER_REGISTRY = {
    "gemma-2b": {
        "base_model": "google/gemma-2b",
        "quantization": "4bit",
        "schools": {
            "nielsen": {
                "local": "adapters/gemma-2b-nielsen-lora",
                "hub": "riaz/designlens-gemma2b-nielsen-lora",
                "description": "Jakob Nielsen's usability heuristics",
            },
            "norman": {
                "local": "adapters/gemma-2b-norman-lora",
                "hub": "riaz/designlens-gemma2b-norman-lora",
                "description": "Don Norman's emotional design",
            },
        }
    },
    "llama3-8b": {
        "base_model": "meta-llama/Meta-Llama-3-8B",
        "quantization": "4bit",
        "schools": {
            "nielsen": {
                "local": "adapters/llama3-8b-nielsen-lora",
                "hub": "riaz/designlens-llama3-nielsen-lora",
                "description": "Jakob Nielsen's usability heuristics",
            },
            "norman": {
                "local": "adapters/llama3-8b-norman-lora",
                "hub": "riaz/designlens-llama3-norman-lora",
                "description": "Don Norman's emotional design",
            },
        }
    },
}

def get_adapter_path(model: str, school: str, prefer_hub: bool = True) -> str:
    """Resolve adapter path from registry."""
    entry = ADAPTER_REGISTRY[model]["schools"][school]
    if prefer_hub:
        return entry["hub"]
    return entry["local"]

def list_available_schools(model: str) -> list:
    """List all available schools for a given model."""
    return list(ADAPTER_REGISTRY[model]["schools"].keys())

def list_available_models() -> list:
    """List all available base models."""
    return list(ADAPTER_REGISTRY.keys())
```

### 8.2 Model Loader + Adapter Swapper

```python
# inference/model_loader.py

"""
Cached model loading with hot-swappable adapters.
Base model loads once, adapters swap instantly.
"""

from peft import PeftModel
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
import torch

class DesignLensEngine:
    def __init__(self):
        self._base_models = {}  # Cache: model_name → (model, tokenizer)
        self._current_adapter = None
        self._current_model = None
    
    def load_base_model(self, model_name: str):
        """Load and cache base model in 4-bit."""
        if model_name not in self._base_models:
            config = ADAPTER_REGISTRY[model_name]
            
            bnb_config = BitsAndBytesConfig(
                load_in_4bit=True,
                bnb_4bit_quant_type="nf4",
                bnb_4bit_compute_dtype=torch.float16,
            )
            
            model = AutoModelForCausalLM.from_pretrained(
                config["base_model"],
                quantization_config=bnb_config,
                device_map="auto",
            )
            tokenizer = AutoTokenizer.from_pretrained(config["base_model"])
            
            self._base_models[model_name] = (model, tokenizer)
        
        return self._base_models[model_name]
    
    def swap_adapter(self, model_name: str, school: str):
        """Hot-swap LoRA adapter."""
        base_model, tokenizer = self.load_base_model(model_name)
        adapter_path = get_adapter_path(model_name, school)
        
        self._current_model = PeftModel.from_pretrained(
            base_model, adapter_path
        )
        self._current_adapter = (model_name, school)
        
        return self._current_model, tokenizer
    
    def generate(self, prompt: str, model_name: str, school: str,
                 max_new_tokens: int = 1024, temperature: float = 0.7):
        """Generate response with specified model and school."""
        
        # Swap adapter if needed
        if self._current_adapter != (model_name, school):
            self.swap_adapter(model_name, school)
        
        model, tokenizer = self._current_model, self._base_models[model_name][1]
        
        formatted = format_inference_prompt(prompt)
        inputs = tokenizer(formatted, return_tensors="pt").to(model.device)
        
        outputs = model.generate(
            **inputs,
            max_new_tokens=max_new_tokens,
            temperature=temperature,
            do_sample=True,
            top_p=0.9,
        )
        
        response = tokenizer.decode(outputs[0], skip_special_tokens=True)
        return extract_response(response)
    
    def compare_all(self, prompt: str, model_name: str):
        """Run prompt through all available schools for comparison."""
        results = {}
        for school in list_available_schools(model_name):
            results[school] = self.generate(prompt, model_name, school)
        return results
```

---

## 9. FRONTEND

### 9.1 Gradio App (Phase 1)

```python
# app/gradio_app.py

import gradio as gr
from inference.model_loader import DesignLensEngine
from inference.adapter_registry import list_available_models, list_available_schools

engine = DesignLensEngine()

def analyze(question: str, model: str, school: str):
    return engine.generate(question, model, school)

def compare_all(question: str, model: str):
    results = engine.compare_all(question, model)
    formatted = ""
    for school, response in results.items():
        formatted += f"## 🔍 {school.title()} School\n\n{response}\n\n---\n\n"
    return formatted

with gr.Blocks(title="DesignLens") as demo:
    gr.Markdown("# 🔍 DesignLens — Multi-Perspective UX Analysis")
    gr.Markdown("Analyze any design problem through different design philosophy lenses.")
    
    with gr.Row():
        model_select = gr.Dropdown(
            choices=list_available_models(),
            value="gemma-2b",
            label="Base Model"
        )
        school_select = gr.Dropdown(
            choices=list_available_schools("gemma-2b"),
            value="nielsen",
            label="Design School"
        )
    
    question = gr.Textbox(
        label="Describe your UX problem",
        placeholder="E.g., Our checkout page has a 68% cart abandonment rate...",
        lines=4
    )
    
    with gr.Row():
        analyze_btn = gr.Button("Analyze", variant="primary")
        compare_btn = gr.Button("Compare All Schools", variant="secondary")
    
    output = gr.Markdown(label="Analysis")
    
    analyze_btn.click(analyze, [question, model_select, school_select], output)
    compare_btn.click(compare_all, [question, model_select], output)

demo.launch()
```

---

## 10. EVALUATION FRAMEWORK

### 10.1 Distinctiveness Test (Critical)

```python
# evaluation/eval_distinctiveness.py

"""
The most important eval: Do different school adapters actually produce
meaningfully different responses?

Method:
1. Run same 50 scenarios through all school adapters
2. Use LLM judge to classify which school wrote each response
3. If classification accuracy > 80%, schools are distinct enough
4. Also measure cosine similarity between school responses (should be low)
"""

def run_distinctiveness_eval(engine, scenarios, schools):
    results = []
    
    for scenario in scenarios:
        responses = {}
        for school in schools:
            responses[school] = engine.generate(scenario, "gemma-2b", school)
        
        # Blind classification test
        for school, response in responses.items():
            predicted = classify_school(response, schools)
            results.append({
                "scenario": scenario,
                "actual_school": school,
                "predicted_school": predicted,
                "correct": school == predicted
            })
    
    accuracy = sum(r["correct"] for r in results) / len(results)
    print(f"School Classification Accuracy: {accuracy:.1%}")
    
    # Target: > 80% accuracy means schools are distinct
    return accuracy, results
```

---

## 11. KEY DECISIONS & RATIONALE

| Decision | Choice | Rationale |
|----------|--------|-----------|
| POC base model | Gemma 2B | Small, fast iteration, JAX-native for TPU |
| Training framework | Unsloth first | 2x speed, well-documented, supports all models |
| Data generation | Synthetic via Claude/GPT-4 | Fastest to quality data, can scale |
| UI framework | Gradio → Streamlit | Gradio fastest for POC, Streamlit for production |
| Adapter format | QLoRA (4-bit NF4) | Best memory efficiency, consumer GPU friendly |
| Hosting | HuggingFace Hub + Spaces | Free, community visibility, standard |
| Evaluation | LLM-as-judge | Scalable, correlates well with human judgment |

---

## 12. ENVIRONMENT SETUP

```bash
# Create environment
conda create -n designlens python=3.11 -y
conda activate designlens

# Core dependencies
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
pip install unsloth
pip install transformers datasets accelerate peft trl
pip install bitsandbytes
pip install gradio

# Data extraction
pip install marker-pdf          # PDF extraction
pip install trafilatura          # Web article extraction
pip install faster-whisper       # Video transcription
pip install beautifulsoup4 requests  # Web scraping
pip install yt-dlp               # YouTube download

# Data generation
pip install anthropic openai     # For synthetic data gen

# Evaluation
pip install scikit-learn numpy pandas

# Optional: JAX for Gemma TPU training
pip install keras keras-nlp jax jaxlib

# Optional: Ollama for local inference
# curl -fsSL https://ollama.ai/install.sh | sh
```

---

## 13. OPEN SOURCE CHECKLIST (Phase 3)

- [ ] LICENSE file (Apache 2.0 or MIT)
- [ ] Comprehensive README with demo GIF
- [ ] Model cards for each adapter (training data, metrics, limitations)
- [ ] Dataset cards for training data
- [ ] CONTRIBUTING.md — how to add a new design school
- [ ] Template: school prompt + training config + eval
- [ ] CI/CD for automated training and eval
- [ ] HuggingFace Collection page
- [ ] Blog post / Twitter thread launch
- [ ] Spaces demo (free, always-on)

---

## 14. CONTEXT FOR CLAUDE

When working on this project, Claude should:

1. **Always reference this CLAUDE.md** for architecture decisions and file locations
2. **Follow the phased approach** — don't jump ahead to Phase 3 features during Phase 1
3. **Prioritize data quality** — the adapters are only as good as the training data
4. **Test distinctiveness early** — if schools don't feel different, revisit data/prompts
5. **Keep adapters small** — the beauty is in the hot-swap, not model size
6. **Think open-source from day 1** — clean code, documentation, reproducibility
7. **Riaz's UX expertise is the moat** — the technical stack is reproducible, the design school knowledge curation is not

---

## 15. QUICK START (Phase 1 — First Session)

```bash
# 1. Setup
git init designlens && cd designlens
# Copy this CLAUDE.md to project root

# 2. Create directory structure
mkdir -p data/{raw,processed,training,scenarios,evaluation}
mkdir -p extraction generation/school_prompts training/configs
mkdir -p inference app evaluation notebooks adapters outputs

# 3. Write school prompts (nielsen.txt, norman.txt)

# 4. Generate scenarios (generate_scenarios.py)

# 5. Generate 500 training pairs per school (generate_pairs.py)

# 6. Quality filter (quality_filter.py)

# 7. Train with Unsloth (train_unsloth.py)

# 8. Evaluate distinctiveness (eval_distinctiveness.py)

# 9. Build Gradio app (gradio_app.py)

# 10. Deploy to HuggingFace Spaces
```

---

*This document is the single source of truth for the DesignLens project. Update it as decisions evolve.*
