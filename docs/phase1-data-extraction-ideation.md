# Phase 1 — Data Extraction Pipeline: Full Ideation

## The Big Picture

Every source type follows the same journey:

**RAW SOURCE -> EXTRACT -> CLEAN/CHUNK -> LLM GENERATES PAIRS -> TRAINING JSONL**

The extracted text does NOT go directly into training.
It either:
- (A) Informs the school system prompts, then LLM generates pairs from scenarios
- (B) Gets used as context for LLM to generate Q&A pairs grounded in the source

---

## Storage Layout

```
data/
  raw/                    <-- Untouched extractions
    books/
      nielsen/            <-- ~50-100 chunks per book
      norman/
    articles/
      nielsen/            <-- ~200 NN/g articles
      norman/             <-- jnd.org posts
    transcripts/
      nielsen/            <-- NN/g talks
      norman/             <-- TED, conferences
    papers/
      nielsen/            <-- ~20-30 papers
      norman/

  processed/              <-- Cleaned + chunked, all types merged
    nielsen/
    norman/

  training/               <-- Final JSONL for fine-tuning
    nielsen_train.jsonl   <-- 500+ pairs
    nielsen_val.jsonl     <-- 50-100 held out
    norman_train.jsonl
    norman_val.jsonl

  scenarios/
    scenarios.json        <-- 200+ reusable UX scenarios

  evaluation/
    distinctiveness_test.jsonl
```

---

## TYPE 1: Book Extraction (PDFs)

### Sources

- Nielsen: "Usability Engineering", "Designing Web Usability"
- Norman: "The Design of Everyday Things", "Emotional Design"

### Tool

Marker (marker-pdf) — best general-purpose PDF to Markdown converter.
Fallback: Docling (IBM) for heavily structured docs.

### Flow

```
PDF file
    |
    v
Marker converts PDF to clean Markdown
    |
    v
Chunker splits by chapter/section headers
(max ~2000 tokens per chunk)
    |
    v
Saved as JSON in data/raw/books/{school}/
```

### Each chunk looks like

```json
{
  "source": "Usability Engineering - Jakob Nielsen",
  "source_type": "book",
  "school": "nielsen",
  "chunk_id": 1,
  "section": "Chapter 3 > Heuristic Evaluation",
  "content": "The actual extracted text...",
  "token_count": 1847
}
```

### Key decisions

- Chunk at section headers, not mid-paragraph
- Keep header hierarchy for context
- You need legal access to the PDFs
- Books are deep knowledge — high signal per chunk

---

## TYPE 2: Website Extraction

### Sources

- Nielsen: nngroup.com/articles/ (2000+ articles, scrape top 200)
- Norman: jnd.org/articles/
- General: Smashing Magazine, A List Apart, Laws of UX

### Tools

- Step 1: requests + BeautifulSoup to crawl article listings
- Step 2: trafilatura to extract clean article text from HTML
- Rate limit: 2-3 seconds between requests

### Flow

```
Article listing page (e.g. nngroup.com/articles/)
    |
    v
Crawl to get all article URLs
    |
    v
For each URL: trafilatura extracts article body
(strips navigation, ads, footer automatically)
    |
    v
Saved as JSON in data/raw/articles/{school}/
```

### Each article looks like

```json
{
  "source_url": "https://www.nngroup.com/articles/ten-usability-heuristics/",
  "source_type": "web_article",
  "school": "nielsen",
  "title": "10 Usability Heuristics for User Interface Design",
  "content": "The full article text...",
  "author": "Jakob Nielsen",
  "date": "1994-04-24",
  "token_count": 2340
}
```

### Key decisions

- trafilatura beats raw BeautifulSoup for article extraction
- NN/g articles are THE gold mine for Nielsen school
- Long articles get chunked in the processing step
- Respect robots.txt always

---

## TYPE 3: YouTube Transcripts

### Sources

- Nielsen: NN/g YouTube channel talks
- Norman: TED talks, conference talks, interviews

### Tools (try in order)

1. youtube-transcript-api (first choice)
   - Pulls existing captions/subtitles
   - Free, instant, no download needed
   - pip install youtube-transcript-api

2. yt-dlp + faster-whisper (fallback)
   - Download audio then transcribe locally
   - Needs GPU, slower, but more accurate
   - Only use if no captions exist

### Flow

```
YouTube URL
    |
    v
Try youtube-transcript-api first
(pulls existing captions — free, instant)
    |
    v  (if no captions exist)
    |
Fallback: yt-dlp downloads audio
faster-whisper transcribes it
    |
    v
Saved as JSON in data/raw/transcripts/{school}/
```

### Each transcript looks like

```json
{
  "source_url": "https://youtube.com/watch?v=...",
  "source_type": "video_transcript",
  "school": "norman",
  "title": "Don Norman - 3 Ways Good Design Makes You Happy",
  "content": "Full transcript text...",
  "duration_seconds": 720,
  "token_count": 3200
}
```

### Key decisions

- Always try caption API first (free and fast)
- Transcripts are noisy — need heavier cleaning
- Conference talks are high signal (thinkers explaining own ideas)
- Prioritize talks BY the thinkers themselves

---

## TYPE 4: Academic Papers

### Sources

- CHI, DIS, NordiCHI conference papers
- Search via Semantic Scholar API (free, no auth for basic use)

### Tools

- Semantic Scholar API for search + metadata
- Marker or Nougat (Meta) for PDF extraction
- Nougat is better for papers with equations/tables

### Flow

```
Search query
(e.g. "usability heuristics evaluation")
    |
    v
Semantic Scholar API returns papers
Filter for open-access PDFs only
    |
    v
Download PDF
Extract with Marker or Nougat
    |
    v
Saved as JSON in data/raw/papers/{school}/
```

### Each paper looks like

```json
{
  "source_url": "https://doi.org/...",
  "source_type": "academic_paper",
  "school": "nielsen",
  "title": "Paper title",
  "authors": ["Author A", "Author B"],
  "abstract": "The abstract...",
  "content": "Full paper body...",
  "year": 2019,
  "token_count": 8500
}
```

### Key decisions

- Papers are supplementary — lowest priority for Phase 1
- Books + web articles carry 80% of the signal
- Only grab open-access PDFs (no paywall issues)

---

## THE PROCESSING STEP (raw -> processed)

All source types flow through the same cleaning pipeline:

```
data/raw/{type}/{school}/*.json
    |
    v
Cleaning:
  - Remove boilerplate, navigation remnants
  - Fix encoding issues
  - Normalize whitespace
  - Remove references/footnotes (for papers)
    |
    v
Chunking (if not already chunked):
  - Split long docs to ~1500-2000 tokens
  - 100-200 token overlap between chunks
  - Preserve section headers as context
    |
    v
data/processed/{school}/*.json
```

---

## HOW PROCESSED DATA BECOMES TRAINING PAIRS

This is the most important step. Two strategies:

### Strategy A: Scenario-Based Generation (Primary — 70% of pairs)

The extracted content informs the system prompts.
The LLM generates pairs from UX scenarios, not directly from chunks.

```
INPUTS:
  - School system prompt (nielsen.txt or norman.txt)
    Written by hand, informed by extracted content
  - UX scenario from scenarios.json
    "Checkout page has 68% cart abandonment..."
  - Question type
    "Analyze this problem" / "How would you redesign?"

PROCESS:
  Send to Claude API / GPT-4:
    System: [school prompt]
    User: [scenario + question]

OUTPUT (one training pair):
  {
    "instruction": "Analyze this UX problem.",
    "input": "Checkout page has 68% abandonment...",
    "output": "From a usability heuristics perspective...",
    "school": "nielsen"
  }
```

Why this works: Same scenario + same question through different
school prompts produces contrastive pairs. The model learns to
"think like Nielsen" vs "think like Norman."

### Strategy B: Content-Grounded Generation (Supplementary — 30%)

Use extracted chunks directly as context for Q&A generation:

```
INPUTS:
  - An extracted chunk from a book/article
    (e.g., a passage from "Design of Everyday Things")

PROCESS:
  Send to Claude API:
    "Given this passage from Don Norman's work,
     generate a UX question a designer might ask
     and answer it in Norman's design philosophy."

OUTPUT (one training pair):
  {
    "instruction": "What makes a door handle confusing?",
    "input": "A glass door has a flat plate on both sides...",
    "output": "This is a signifier-affordance mismatch...",
    "school": "norman"
  }
```

Why this works: Grounds the training in actual source material.
Adds depth and faithfulness to the school's real ideas.

---

## FINAL TRAINING FORMAT (what the model actually sees)

Each line in the JSONL file:

```json
{
  "instruction": "The question type or task",
  "input": "The UX scenario or context",
  "output": "The school-specific analysis response",
  "school": "nielsen",
  "metadata": {
    "generation_model": "claude-sonnet",
    "scenario_domain": "ecommerce",
    "question_type": "analysis",
    "source_strategy": "scenario_based"
  }
}
```

These get formatted into the training template:

```
### Instruction:
{instruction}

### Input:
{input}

### Response:
{output}
```

The model learns: given an instruction + input, produce a
response in the style of the specified school.

---

## PRIORITY ORDER FOR PHASE 1

| # | Source Type              | Why                                    | Effort      |
|---|-------------------------|----------------------------------------|-------------|
| 1 | Scenario-based gen      | 500 pairs fastest, no extraction needed | Low         |
| 2 | Web scraping (NN/g)     | Highest signal per effort              | Medium      |
| 3 | Book extraction         | Deep knowledge, needs PDFs             | Medium-High |
| 4 | YouTube transcripts     | Good supplementary, noisy text         | Medium      |
| 5 | Academic papers         | Lowest priority for POC                | Low priority |

### Recommended Phase 1 approach

1. Start with Priority 1: pure synthetic from scenarios (get POC working)
2. Layer in Priority 2-3 to improve quality before final training
3. Skip 4-5 unless you have extra time

---

## QUALITY GATE

Before training, all pairs go through LLM-as-judge scoring:

- School Faithfulness (1-5): Does it follow the school's principles?
- Distinctiveness (1-5): Could you tell which school wrote it?
- Quality (1-5): Is the analysis useful and specific?
- Consistency (1-5): Does it use the right vocabulary?

Discard pairs with average score below 3.5.
Target: 500+ surviving pairs per school after filtering.
