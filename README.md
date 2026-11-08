# LLM-Prompt-Evaluator-Pipeline

Evaluates **prompt versions**, not models. The prompt is the only variable. Model, temperature, test data and judge stay fixed.

---

## Repo structure

```
LLM-Prompt-Prompt-Evaluator-Pipeline/
├── README.md
├── requirements.txt
├── results/                          # run outputs + metadata (gitignored)
├── .github/workflows/                # smoke run on PR, full run on merge
│
└── src/
    ├── configs/                      # WHAT to run: one promptfoo config per use case
    │
    ├── prompts/                      # Step 1: prompt versions, never overwrite old ones
    │
    ├── context/                      # domain knowledge passed to the prompt as a fixed variable
    │
    ├── data/                         # Step 2: test data
    │
    ├── runner/                       # Step 3: execute prompts against the data
    │
    ├── graders/                      # Step 4: scoring methods
    │
    ├── calibration/                  # Step 4: how far can the judge be trusted?
    │
    ├── compare/                      # Step 5: compare and decide
    │
    ├── gates/                        # gate rules per use case
    │
    ├── baselines/                    # saved reference scores (committed)
    │
    ├── feedback/                     # Step 6: close the loop
    │   └── promote_failures.py       #   failed cases -> data/<usecase>/regressions.yaml
    │
    └── scripts/
```

## Workflow in 6 steps

| # | Step | What happens |
|---|---|---|---|
| 1 | Draft a prompt | New version as a new file |
| 2 | Create data and criteria | Golden + edge cases, rubric, critical categories |
| 3 | Run through the model | Fixed model, temperature 0, repeats, save metadata |
| 4 | Grade | Deterministic checks first, then judge. Calibrate the judge once against your own labels |
| 5 | Compare and gate | Against baseline, per category and per case, tolerance bands |
| 6 | Change and repeat | New failures into regressions, winner becomes baseline |

---

## Grading methods

| Method | Measures | Strength | Weakness |
|---|---|---|---|
| Rules (contains, regex, custom JS) | Clear violations | Cheap, stable | Pattern only |
| Metrics (length, tokens, latency, cost) | Answer shape and cost | Objective | Says nothing about quality |
| Number grounding | Invented values | Deterministic factuality | Numbers only |
| Judge, binary rubric | Criteria met yes/no | Flexible, explainable | Judge can be wrong |
| Judge, scale 1-10 | Degree of quality | Finer differences | Noisy, tends to favour longer answers |
| Pairwise A/B | Which version is better | More robust than absolute scores | Relative only, double cost |
| Consistency across repeats | Unstable cases | Exposes randomness | Needs several runs |
| Human calibration | How far to trust the judge | Makes scores credible | Manual work (about 20-30 cases) |
| Reference overlap (informational only) | Similarity to a sample answer | Quick signal | Punishes good answers phrased differently, never use as pass criterion |

## Data sources (e-commerce)

| Purpose | Source |
|---|---|
| Support Q&A | Bitext Customer Support (Hugging Face `bitext/Bitext-customer-support-llm-chatbot-training-dataset`) |
| Product questions | Amazon Question/Answer Data (UCSD, per category `qa_*.json.gz`) plus product metadata (`meta_*.json.gz`) |

Answers from these sources are references only. Grading is rubric-based. Check licences before redistributing (Bitext: CDLA-Sharing-1.0).