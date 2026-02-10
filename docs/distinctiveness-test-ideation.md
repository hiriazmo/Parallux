# Distinctiveness Test: Full Ideation

## What Is This Test?

The cheapest, fastest way to validate your entire project premise:
**Can two different system prompts make an LLM reason
about the SAME design problem in genuinely different ways?**

If YES --> fine-tuning will amplify this. Proceed with confidence.
If NO  --> fix the prompts first. No point training adapters.
If MAYBE --> the prompts need sharpening. Iterate before training.

---

## The Setup

```
                   Scenario #1
                 "Checkout has 68%
                  abandonment rate"
                       |
            +----------+----------+
            |                     |
     Nielsen Prompt          Norman Prompt
     (nielsen.txt)           (norman.txt)
            |                     |
            v                     v
     Response A              Response B
     "Heuristic #3 is        "The visceral reaction
      violated. Error         to the payment form
      prevention score..."    creates anxiety..."
            |                     |
            +----------+----------+
                       |
                  BLIND JUDGE
              "Which school wrote
               Response A vs B?"
                       |
                  Classification
                  Accuracy: ??%
```

---

## Step-by-Step Design

### STEP 1: Select 20 Scenarios

Why 20? Enough to be statistically meaningful, cheap to run (~$2-5).

**Selection criteria:**
- Cover different domains (not all e-commerce)
- Mix of problem types (analysis, redesign, critique)
- Vary in complexity (simple UI issue vs systemic problem)
- Include some that SHOULD trigger different school responses
- Include some "trap" scenarios where schools might converge

**The 20 Scenarios:**

DOMAIN: E-COMMERCE (4 scenarios)
  1. Checkout page has 68% cart abandonment rate.
     Users add items but don't complete purchase.

  2. Product page has a 5-star rating display but
     conversion is low despite high ratings.

  3. Mobile product listing has 3x higher bounce
     rate than desktop version.

  4. Size selector on clothing site causes 40%
     of customer support tickets.

DOMAIN: SAAS (4 scenarios)
  5. SaaS onboarding flow has 12 steps and only
     23% completion rate.

  6. Dashboard has 15 widgets but analytics show
     users only interact with 3 of them.

  7. Settings page receives the highest rage-click
     count in the entire application.

  8. Users complete tasks in the app but report
     low satisfaction in NPS surveys.

DOMAIN: HEALTHCARE (3 scenarios)
  9. Patient portal appointment booking takes an
     average of 4.5 minutes to complete.

 10. Elderly users abandon telehealth setup at
     the camera/microphone permission step.

 11. Lab results page causes the highest support
     call volume in the patient portal.

DOMAIN: FINTECH (3 scenarios)
 12. Banking app money transfer has higher error
     rate on mobile than desktop.

 13. Investment portfolio page shows too much data.
     Users miss critical alerts and notifications.

 14. Budget tracking feature has high initial setup
     completion but very low daily return rate.

DOMAIN: GENERAL UX (4 scenarios)
 15. Navigation has 40+ flat items. Users can't
     find what they're looking for.

 16. A/B test shows redesign increases conversion
     by 15% but decreases satisfaction by 20%.

 17. Design system components aren't being adopted
     by the development teams.

 18. Error messages across the app are technical
     and use system codes instead of plain language.

DOMAIN: EDGE CASES - "Trap" scenarios (2 scenarios)
 19. Login page has a password field that doesn't
     show/hide toggle and no "forgot password" link.
     (Both schools should flag this, but differently)

 20. A museum exhibit app uses haptic feedback and
     ambient sound to guide visitors through galleries.
     (Should strongly favor Norman's emotional lens)

---

### STEP 2: The School Prompts

These already exist in your CLAUDE.md. Key difference between them:

NIELSEN focuses on:
  - Which of the 10 heuristics are violated
  - Severity rating (0-4 scale)
  - Measurable outcomes (task time, error rate, completion rate)
  - Evidence-based recommendations
  - Vocabulary: heuristic, learnability, efficiency, error prevention

NORMAN focuses on:
  - Visceral/Behavioral/Reflective emotional response
  - Affordance/signifier/mapping/feedback breakdown
  - Gulf of execution and gulf of evaluation
  - How the design makes users FEEL
  - Vocabulary: affordance, signifier, conceptual model, visceral

The prompts should be STRICT. The more prescriptive they are about
reasoning pattern and vocabulary, the more distinct the outputs.

---

### STEP 3: Generate Responses

For each of the 20 scenarios, generate TWO responses:
  - One with nielsen.txt as system prompt
  - One with norman.txt as system prompt

Total: 40 API calls.

**Important parameters:**
  - temperature: 0.7 (some creativity, not too random)
  - max_tokens: 800 (enough depth, not rambling)
  - model: claude-sonnet (or whatever you'll use for data gen)

**The user message format:**
  "Analyze this UX problem and provide your assessment.
   Be specific, actionable, and demonstrate your school
   of thought's unique reasoning pattern.

   Problem: {scenario}"

---

### STEP 4: Blind Judging

This is where the magic happens. Three types of evaluation:

#### Judge Type A: School Classification (Core Metric)

Present each response WITHOUT telling the judge which school wrote it.
Ask it to classify.

JUDGE PROMPT:
  "You are an expert in UX design philosophies.
   Below is a UX analysis response. Based on the reasoning
   style, vocabulary, and framework used, classify which
   design school wrote this response.

   Options:
   A) Nielsen School (usability heuristics, task efficiency,
      severity ratings, measurable outcomes)
   B) Norman School (emotional design, affordances/signifiers,
      visceral-behavioral-reflective, conceptual models)

   Response to classify:
   {response}

   Answer with just the school name and a 1-sentence explanation
   of why you chose it."

METRIC: Classification accuracy
  TARGET: > 80% = schools are distinct enough
          60-80% = prompts need sharpening
          < 60% = fundamental problem, rethink approach

#### Judge Type B: Paired Comparison (Depth Metric)

Show BOTH responses side by side for the SAME scenario.
Ask the judge to identify differences.

JUDGE PROMPT:
  "Below are two UX analyses of the same problem,
   written by analysts from different design schools.

   Response A: {nielsen_response}
   Response B: {norman_response}

   Score the following (1-5 each):

   1. REASONING_DIFFERENCE: Do they use fundamentally
      different reasoning frameworks?
      1=identical reasoning, 5=completely different logic

   2. VOCABULARY_DIFFERENCE: Do they use different
      terminology and jargon?
      1=same words, 5=clearly different vocabularies

   3. RECOMMENDATION_DIFFERENCE: Do they suggest
      different solutions or priorities?
      1=same fixes, 5=totally different priorities

   4. COULD_BE_SWAPPED: If you swapped the labels,
      would it still make sense?
      1=easily swappable (bad), 5=impossible to swap (good)

   Respond as JSON with scores and brief reasoning."

METRIC: Average scores across all 20 scenarios
  TARGET: All dimensions averaging > 3.5

#### Judge Type C: Human Spot Check (Reality Check)

YOU (Riaz) read 5 paired responses yourself. As a UX expert, ask:
  - Does the Nielsen response sound like Nielsen would think?
  - Does the Norman response sound like Norman would think?
  - Are the differences meaningful or superficial?
  - Would a UX professional find both analyses useful?

No metric. Just gut check from the domain expert.

---

### STEP 5: Analyze Results

#### What Success Looks Like

Classification accuracy: 85-95%
  - Nielsen responses consistently flagged as Nielsen
  - Norman responses consistently flagged as Norman
  - Only 1-3 misclassifications out of 40

Paired comparison scores:
  - Reasoning difference: 4.0+
  - Vocabulary difference: 4.5+ (this should be easiest)
  - Recommendation difference: 3.5+
  - Could be swapped: 4.0+

Human spot check:
  - "Yes, these feel like different experts talking"

Example of GOOD distinctiveness:

  Scenario: "Error messages use system codes"

  Nielsen would say:
    "This violates Heuristic #9: Help users recognize,
     diagnose, and recover from errors. Severity: 3
     (major). Error messages should use plain language,
     precisely indicate the problem, and constructively
     suggest a solution. Current error rate likely
     increases task abandonment by 30-40%."

  Norman would say:
    "The technical error messages create a gulf of
     evaluation — users cannot form a conceptual model
     of what went wrong. At the visceral level, seeing
     'ERR_4032' triggers anxiety and helplessness. The
     system's image doesn't match the user's mental model.
     Users need signifiers that map to their understanding,
     not the developer's."

Both say "fix the error messages" but the REASONING is different.
That's what we're testing for.

#### What Failure Looks Like

Classification accuracy: 50-60%
  - Responses are generic, could be from either school
  - Both say the same things with slightly different words

Example of BAD distinctiveness:

  Nielsen: "The error messages are confusing and should
           be clearer for better user experience."

  Norman: "The error messages don't provide good feedback
          and should be improved for the user."

These are the same response wearing different hats.
If this happens, the prompts need to be more prescriptive.

---

### STEP 6: What To Do With Results

IF ACCURACY > 80%:
  Great. Proceed to training data generation.
  The prompts work. Fine-tuning will amplify the signal.

IF ACCURACY 60-80%:
  The prompts need sharpening. Common fixes:
  1. Add MORE specific vocabulary lists to each prompt
  2. Add "YOU ALWAYS start your analysis with..." directives
  3. Add "YOU NEVER discuss..." exclusion rules
  4. Add example responses in the prompt (few-shot)
  5. Make the reasoning STEPS more explicit and different
  Iterate and re-test until > 80%.

IF ACCURACY < 60%:
  Fundamental problem. Options:
  1. The schools are too similar (unlikely for Nielsen vs Norman)
  2. The prompts are too vague (most likely cause)
  3. The scenarios don't trigger school-specific thinking
  4. Rewrite prompts from scratch with harder constraints

---

## Cost & Time Estimate

API calls:
  - 40 generation calls (20 scenarios x 2 schools)
  - 40 classification calls (Judge Type A)
  - 20 paired comparison calls (Judge Type B)
  - Total: 100 API calls

Using Claude Sonnet:
  - ~800 input tokens + ~800 output tokens per call
  - 100 calls x 1600 tokens = ~160K tokens
  - Cost: roughly $2-5

Time:
  - Write the script: 1-2 hours
  - Run the test: 5-10 minutes
  - Analyze results: 30 minutes
  - Total: half a day

---

## What This Test Tells You About the Whole Project

| Test Result | Implication |
|-------------|-------------|
| Prompts alone score 90%+ | FT will work great. Strong signal to amplify. |
| Prompts alone score 80-90% | FT is justified. It'll push above 90%. |
| Prompts alone score 60-80% | Fix prompts first. FT can't fix bad prompts. |
| Prompts alone score < 60% | Rethink the school definitions entirely. |
| Nielsen strong, Norman weak | Norman prompt needs more work. |
| Norman strong, Nielsen weak | Nielsen prompt needs more work. |
| Both strong but similar recs | Add recommendation constraints to prompts. |
| Vocabulary distinct, logic same | Need different REASONING STEPS, not just words. |

---

## Bonus: What This Test Produces As Side Effects

Even if you just run this test, you get:
  1. 40 high-quality responses (20 per school) that become SEED training data
  2. Validated school prompts ready for bulk generation
  3. A working evaluation script reusable for Phase 1 eval
  4. Confidence (or course correction) before investing in training

This is why it's the #1 thing to do first.
Maximum learning per dollar spent.
