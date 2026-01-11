# PROJECT PLAN: MULTI-AGENT EVALUATOR-OPTIMIZER PIPELINE

## 1. MISSION STATEMENT
To iteratively evolve an AI Writer Agent's capability through a weighted, human-in-the-loop genetic selection process.

## 2. THE AGENT ARCHITECTURE
- AGENT 1 (Reviewer One)
- AGENT 2 (Reviewer Two)
- AGENT 3 (Reviewer Three)
- AGENT 4 (The Writer): The execution agent whose system prompt is updated iteratively.

## 3. THE WORKFLOW (PER ITERATION)
1. GENERATION: The Writer Agent generates 10 essays on 10 pre-defined subjects.
2. AI EVALUATION: Agents 1, 2, and 3 score all 10 essays using the Shared Rubric.
3. RANKING: System identifies the Top 2 essays based on AI average scores.
4. HUMAN REVIEW: The user reviews the Top 2 and provides a score and notes.
5. WEIGHTED SCORING:
   Final_Score = ((Agent1 + Agent2 + Agent3) + (Human_Score * 3)) / 6
6. META-OPTIMIZATION: The Lead Writer Evaluator and Project Manager analyzes the AI and Human reviews and the weighted average score. It generates a "Context Patch" (a revised System Prompt) for the Writer to use in the next iteration.

## 4. BEST PRACTICES
- MANDATORY COT: Critics must provide reasoning BEFORE scores to ensure logic.
- VERSIONING: Each "Context Patch" should be saved as a new version (e.g., Writer_System_Prompt_V2)