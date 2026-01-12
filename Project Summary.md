# PROJECT PLAN: MULTI-AGENT EVALUATOR-OPTIMIZER PIPELINE

## 1. MISSION STATEMENT
To iteratively evolve an AI Writer Agent's capability through a weighted, human-in-the-loop genetic selection process. The system uses three calibrated AI reviewers with distinct stylistic preferences and a human reviewer to identify quality gaps, then synthesizes feedback into actionable style guidelines that improve essay quality over successive iterations.

---

## 2. THE AGENT ARCHITECTURE

### Writer Agent
- **Role**: Produces 10 essays per iteration on pre-2000 historical topics
- **Constraints**: 800-1500 words per essay, no internet sources
- **Evolution**: System prompt's "CURRENT STYLE GUIDELINES" section is revised each iteration based on reviewer feedback
- **Prompt Location**: `.claude/agents/writer.md`

### Reviewer Agents (1-3)
Each reviewer is calibrated with distinct writing samples that shape their evaluation lens:

| Agent | Focus | Calibration Samples | Values |
|-------|-------|---------------------|--------|
| **Reviewer One** | Clear, informative, convincing | Academic analysis (Panopticon + Chainsaw Man) | Intellectual rigor, structured argumentation, analytical depth |
| **Reviewer Two** | Engaging, logical, plainspoken | Russell's philosophical essays, SNAP policy analysis | Logical depth, plainspoken sophistication, practical reasoning |
| **Reviewer Three** | Clever, engaging, persuasive | Political journalism (Manchin), nationalism essays | Storytelling, data-driven wit, persuasive rhetoric |

### Reviewer Output Format
Each reviewer provides:
- Chain of Thought (CoT) reasoning before scoring
- JSON output with: essay_id, criteria scores (1-5 each), overall score, notes

---

## 3. THE SCORING SYSTEM

### Rubric (4 Criteria, 1-5 Scale Each)
1. **Engagement**: Is the essay engaging?
2. **Thesis & Logic**: Is the thesis clear and supported by strong logic/reasoning?
3. **Writing Style**: Does the essay have good writing style?
4. **Enjoyment**: Do you enjoy the essay?

**Maximum Score per Reviewer**: 20 points

### Weighted Final Score Formula
```
Final_Score = ((Agent1 + Agent2 + Agent3) + (Human_Score * 3)) / 6
```
- AI Reviewers collectively = 50% weight
- Human Reviewer = 50% weight

---

## 4. THE WORKFLOW (PER ITERATION)

```
┌─────────────────────────────────────────────────────────────────┐
│  ITERATION N                                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. GENERATION                                                  │
│     └── Writer Agent generates 10 essays                        │
│                        ↓                                        │
│  2. AI EVALUATION                                               │
│     └── Reviewers 1, 2, 3 score all 10 essays (CoT + JSON)      │
│                        ↓                                        │
│  3. RANKING                                                     │
│     └── Calculate AI average scores,                            │
│       identify 4th best and 5th best essays.                    │
│                        ↓                                        │
│  4. HUMAN REVIEW                                                │
│     └── Human reviews 4th/5th best essays,                      │
│          provides scores + notes                                │
│                        ↓                                        │
│  5. WEIGHTED SCORING                                            │
│     └── Calculate final weighted scores for 4th/5th essays      │
│                        ↓                                        │
│  6. META-OPTIMIZATION                                           │
│     └── Analyze feedback, generate revised Style Guidelines     │
│                        ↓                                        │
│  7. CONTEXT PATCH                                               │
│     └── Update Writer's prompt with new guidelines              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                        ↓
              ITERATION N+1 (repeat)
```

---

## 5. IMPLEMENTATION PLAN

### Phase 1: Essay Generation
**Objective**: Produce 10 essays for evaluation

**Steps**:
1. Invoke Writer Agent with current style guidelines
2. Writer generates 10 essays on distinct pre-2000 historical topics
3. Save essays to `Essay Evaluation Folder/` with clear labeling (Essay 1, Essay 2, etc.)

**Output**: 10 essays in markdown format

---

### Phase 2: AI Review Collection
**Objective**: Gather structured evaluations from all three reviewer agents

**Steps**:
1. Invoke Reviewer One Agent → Review all 10 essays
2. Invoke Reviewer Two Agent → Review all 10 essays
3. Invoke Reviewer Three Agent → Review all 10 essays
4. Collect JSON outputs with scores and notes for each essay

**Output**: 30 review records (10 essays × 3 reviewers)

**JSON Schema per Review**:
```json
{
  "essay_id": 1,
  "engagement": 4,
  "thesis_logic": 3,
  "writing_style": 4,
  "enjoyment": 3,
  "overall_score": 14,
  "notes": "Detailed feedback..."
}
```

---

### Phase 3: Ranking & Selection
**Objective**: Identify 4th and 5th best essays for human review

**Steps**:
1. Calculate average AI score for each essay:
   `AI_Average = (R1_Score + R2_Score + R3_Score) / 3`
2. Rank all 10 essays by AI average (descending)
3. Select 4th and 5th best essays for human review
4. Present 4th and 5th best essays to human reviewer with AI scores summary

**Output**: Ranked list, 4th and 5th best essays flagged for human review

---

### Phase 4: Human Review
**Objective**: Collect human evaluation of top essays

**Steps**:
1. Present 4th and 5th best essays to human reviewer
2. Human scores each essay using same rubric (4 criteria, 1-5 scale)
3. Human provides written notes on strengths/weaknesses
4. Record human scores and notes

**Output**: Human scores and qualitative feedback for 4th and 5th best essays

---

### Phase 5: Weighted Score Calculation
**Objective**: Calculate final weighted scores

**Steps**:
1. For each of 4th and 5th best essays, apply formula:
   ```
   Final_Score = ((R1 + R2 + R3) + (Human × 3)) / 6
   ```
2. Record final weighted scores
3. Compare to previous iteration's scores (if applicable)

**Output**: Final weighted scores, iteration-over-iteration comparison

---

### Phase 6: Feedback Analysis
**Objective**: Synthesize all reviewer feedback into actionable insights

**Steps**:
1. **Aggregate Notes**: Compile all reviewer notes (AI + Human) for 4th and 5th best essays
2. **Pattern Identification**: Identify recurring themes across reviewers
   - What do all reviewers praise?
   - What do all reviewers criticize?
   - Where do reviewers diverge?
3. **Gap Analysis**: Compare writer output against each reviewer's calibration samples
   - Is the writing rigorous enough for Reviewer One?
   - Is it plainspoken enough for Reviewer Two?
   - Is it clever/persuasive enough for Reviewer Three?
4. **Priority Ranking**: Rank improvement opportunities by potential score impact

**Output**: Synthesized feedback report with prioritized improvement areas

---

### Phase 7: Style Guidelines Revision (Context Patch)
**Objective**: Rewrite Writer's "CURRENT STYLE GUIDELINES" to address feedback

**Steps**:
1. Draft new style guidelines addressing top improvement priorities
2. Include actionable constraints (not vague suggestions)
3. Balance competing reviewer preferences (rigor vs. accessibility vs. wit)
4. Version the new guidelines (e.g., V2, V3)
5. Update `.claude/agents/writer.md` with new guidelines

**Guidelines Structure**:
```markdown
## CURRENT STYLE GUIDELINES (V[N])

### Voice & Tone
[Specific guidance on narrative voice, formality level, wit usage]

### Argument Structure
[Guidance on thesis placement, evidence integration, logical flow]

### Engagement Techniques
[Guidance on hooks, storytelling elements, reader connection]

### Style Constraints
[Word count, paragraph structure, vocabulary level]
```

**Output**: Revised Writer prompt ready for next iteration

---

### Phase 8: Iteration Tracking
**Objective**: Maintain records for measuring improvement

**Metrics to Track**:
- Average AI score per iteration
- Average weighted score per iteration
- Score distribution across rubric criteria
- Reviewer agreement/divergence patterns

**Storage**:
- Save each iteration's data in `Essay Evaluation Folder/Iteration_N/`
- Include: essays, reviews, scores, style guidelines version used

---

## 6. BEST PRACTICES

### Evaluation Integrity
- **MANDATORY COT**: Reviewers must provide reasoning BEFORE scores to ensure logical consistency
- **BLIND REVIEW**: Reviewers should not see other reviewers' scores before completing their own
- **CONSISTENT RUBRIC**: All reviewers use identical 4-question rubric

### Versioning & Traceability
- Save each Context Patch as a new version (e.g., `Writer_Style_Guidelines_V2.md`)
- Document what changes were made and why
- Link each iteration's essays to the style guidelines version that produced them

### Balancing Reviewer Preferences
The three reviewers have intentionally different preferences. Effective style guidelines must find synthesis:
- **Rigor + Accessibility**: Be intellectually substantive without being dense
- **Logic + Narrative**: Structure arguments clearly while maintaining storytelling flow
- **Sophistication + Plainspokenness**: Use precise language without unnecessary jargon
- **Persuasion + Honesty**: Be compelling without being manipulative

---

## 7. SUCCESS CRITERIA

**Primary Metric**: Average weighted score improvement per iteration

**Secondary Metrics**:
- Reduction in score variance across reviewers (indicates balanced appeal)
- Improvement in lowest-scoring rubric criterion
- Positive trend in human reviewer scores

**Target**: Continuous improvement in weighted scores until plateau is reached

---

## 8. FILE STRUCTURE

```
writer-evaluation/
├── CLAUDE.md                          # Project manager instructions
├── Project Summary.md                 # This document
├── Reviewer Rubric.md                 # Scoring criteria
├── .claude/agents/
│   ├── writer.md                      # Writer agent prompt
│   ├── reviewer_one.md                # Reviewer 1 prompt
│   ├── reviewer_two.md                # Reviewer 2 prompt
│   └── reviewer_three.md              # Reviewer 3 prompt
├── Reviewer One Writing Samples/      # Calibration samples for R1
├── Reviewer Two Writing Samples/      # Calibration samples for R2
├── Reviewer Three Writing Samples/    # Calibration samples for R3
└── Essay Evaluation Folder/           # Generated essays and reviews
    ├── Iteration_1/
    │   ├── essays/
    │   ├── reviews/
    │   └── scores_summary.md
    └── Iteration_N/
```

---

## 9. IMPLEMENTATION SUMMARY (Iterations 1-3)

### Progress Overview

| Iteration | Weighted Score | Change | Guidelines Version |
|-----------|----------------|--------|-------------------|
| 1 (Baseline) | 8.00 / 20 | — | V1 |
| 2 | 8.25 / 20 | +0.25 | V2 |
| 3 | 8.75 / 20 | +0.50 | V3 |
| **Cumulative** | — | **+0.75 (+9.4%)** | V4 (current) |

### Key Guideline Evolution

**V1 → V2**: Addressed structural issues
- Eliminated "it didn't do X, it did Y" sentence patterns
- Reduced list structures ("X, Y, and Z")
- Required flowing introductions (narrative, not fact-lists)
- Added transitions between sections

**V2 → V3**: Improved argumentation
- Required complete logical explanations with no gaps
- Sentences must flow and connect to each other
- Conclusions must synthesize, not summarize
- Historical context required for criticisms

**V3 → V4**: Fixed prose quality
- ACTIVE VOICE required (especially intros/conclusions)
- Eliminated "AI slop" meta-sentences
- Shorter sentences (7-23 words, avg under 21)
- Section headers required
- Compelling thesis requirement

### What Improved Scores
- Bold, clear thesis statements
- Use of counterfactuals and contrast
- Complete logical arguments
- Compelling paradoxes (success breeding failure, etc.)

### What Hurt Scores
- Passive voice (especially in intros/conclusions)
- Long meta-sentences ("raising fundamental questions...")
- Missing section headers
- Incomplete logical explanations
- Repetitive sentence structures

---

## 10. CURRENT STATUS

**Iteration**: 3 (Complete)
**Essays Generated**: 30 (10 per iteration)
**Style Guidelines Version**: V4
**Weighted Score**: 8.75 / 20 (43.75%)
**Next Action**: Begin Iteration 4 with V4 guidelines

