# ## Architecture & System Specification: Provenance Guard

## 1. ## Architecture Section
When a platform client sends content to Provenance Guard, it follows a structured lifecycle across our isolated backend components. The submission flow validates user inputs against an IP rate limiter before passing text parallelly to a dual-signal extraction pipeline, mathematical weights combine these signals into an ensemble score, and a mapping layer formats plain-language strings before filing data inside an immutable SQLite audit history. The appeal flow reads existing submission states, switches operational status tracking parameters to an under-review flag, and writes content snapshots directly to the log table to await human evaluation.

### Flow Architecture Diagrams
```text
[SUBMISSION FLOW]
POST /api/submit 
       |
       v
 [Rate Limiter] ---> (Hits Limit) ---> Returns HTTP 429 Error
       | (Under Limit)
       v
 [Dual-Signal Pipeline]
       |---> Signal 1: Groq LLM Classifier  ---> Output: Float (0.0 to 1.0) ---\
       |---> Signal 2: Length Variance Heuristic -> Output: Float (0.0 to 1.0) ---> [Scoring Engine]
                                                                                            |
                                                                                    Final Combined Score
                                                                                            |
                                                                                            v
                                                                                    [UX Mapping Layer]
                                                                                            |
                                                                                  Plain Language Label
                                                                                            |
                                                                                            v
                                                                                  [SQLite Database Log]
                                                                                            |
                                                                                  Returns HTTP 201 JSON

[APPEAL FLOW]
POST /api/submit/<id>/appeal
       |
       v
 [SQLite State Lookup] ---> (Record Missing) ---> Returns HTTP 404 Error
       | (Record Found)
       v
 [Database Transaction] ---> Updates status column flag to 'under_review'
       |
       v
 [Audit Log Injection]  ---> Inserts creator reasoning snapshot + timestamp
       |
       v
 Returns HTTP 200 JSON Response
```

---

## 2. Technical Specification & Design Questions

### A. Detection Signals
Our backend utilizes a multi-signal ensemble framework consisting of two completely distinct analytical vectors:
1. **Signal 1: Groq LLM Semantic Classifier**
   * *What it measures*: Phrase structural predictability, semantic smoothing, and token distribution transition density using the `llama-3.3-70b-versatile` model.
   * *Output profile*: A singular normalized raw float between `0.0` (Highly human signature) and `1.0` (Highly predictable machine pattern).
2. **Signal 2: Stylometric Sentence Length Variance**
   * *What it measures*: Pure-Python algorithmic metric tracking the mathematical variance (spread) of word counts per sentence across the body text.
   * *Output profile*: A normalized raw float between `0.0` (Dynamic variation / biological rhythm) and `1.0` (Uniform structural lengths / machine pacing).

#### Signal Combination Formula
The outputs are combined using a weighted average that prioritizes semantic holistic structure while keeping an independent structural checkpoint:
$$\text{Final Combined Score} = (\text{Groq Score} \times 0.60) + (\text{Variance Score} \times 0.40)$$

### B. Uncertainty Representation & Thresholds
A score of `0.60` represents a borderline classification in our system. It indicates that while some elements appear highly uniform or mechanical, other attributes display natural human variations. Rather than forcing a binary flip at 0.50, our system uses asymmetric guard rails to protect human creators from aggressive false positives:
* **0.00 to 0.40 (High-Confidence Human)**: Text matches biological writing parameters with high variance.
* **0.41 to 0.74 (Uncertain / Mixed Provenance)**: Flagged for borderline structural metrics; routes to a protective warning label rather than an outright automated penalty.
* **0.75 to 1.00 (High-Confidence AI)**: Overwhelming markers match generative patterns across both signals.

### C. Transparency Label Design
The system uses the following plain-language text variants:
* **High-Confidence Human Variant (`0.00 - 0.40`)**: `"Verified Original: Our system confirms this work matches natural human writing patterns and structural variety."`
* **Uncertain / Mixed Variant (`0.41 - 0.74`)**: `"Mixed Structural Patterns: This text exhibits a combination of predictable phrasing and unique structures. It may contain co-authored or heavily edited content."`
* **High-Confidence AI Variant (`0.75 - 1.00`)**: `"AI-Generated Patterns: Analysis indicates highly uniform sentence structures and vocabulary choices typical of automated writing tools."`

### D. Appeals Workflow
* **Who can appeal**: Registered creators who have active content entries saved in our backend database.
* **Information provided**: A text string field containing the creator's justification statement (`reasoning`).
* **System Operations**: The backend opens a transaction to SQLite, checks if the item exists, mutates the submission state column flag from `classified` to `under_review`, and injects an un-mutable payload block containing their arguments, timestamps, and previous metrics directly into the audit history log.
* **Moderator Queue View**: A human moderator opening the dashboard sees a relational table highlighting all records marked `under_review`, presenting the creator's defense text side-by-side with the historical signal metrics (`groq_llm_score` vs `sentence_length_variance_score`) and original timestamps.

### E. Anticipated Edge Cases & Structural Failures
1. **Edge Case 1: Legal Disclaimers & Technical Privacy Documents**
   * *Failure Mechanism*: Human attorneys must use repetitive, uniform phrase layouts and highly predictable compliance vocabulary (low burstiness and low lexical variation). The pipeline will misclassify this formal human writing as machine-generated text.
2. **Edge Case 2: Minimalist Free-Verse Creative Poetry**
   * *Failure Mechanism*: Human poets writing avant-garde verse often use deliberately flat, short, three-word sentences across lines to set a specific tone. Because sentence length variance drops toward zero, our stylometric heuristic will generate an artificial `1.0` AI score, potentially dragging an original human poem into a false-positive flag.

---

## 3. ## AI Tool Plan Section

### Milestone 3 (Submission Endpoint & First Signal)
* **Spec Sections Provided**: Section 1 (Diagrams) + Section 2A (Signal 1 Profile).
* **AI Request Prompt**: "Generate a synchronous Flask app skeleton with an initial placeholder database schema alongside a Python function that uses the groq client library to classify input text syntax via a JSON output constraint format."
* **Verification Method**: Pass a hardcoded mock context payload string through the runtime and inspect that a JSON dictionary is returned with the correct status types.

### Milestone 4 (Second Signal & Confidence Scoring)
* **Spec Sections Provided**: Section 1 (Diagrams) + Section 2A (Formula) + Section 2B (Threshold Layout).
* **AI Request Prompt**: "Write a pure-Python string processing method that tokenizes sentences by punctuation markers, measures their word length variances, scales the outputs between 0.0 and 1.0, and executes our weighted fusion formula."
* **Verification Method**: Pass an ultra-repetitive block of text versus a variable narrative passage to verify that the math formula yields distinct numerical changes.

### Milestone 5 (Production Layer)
* **Spec Sections Provided**: Section 2C (Label Text) + Section 2D (Appeals Parameters).
* **AI Request Prompt**: "Build a Flask decorator routing system that implements rate limiting, maps our conditional boundary scales onto verbatim text string labels, handles SQLite under_review mutations, and serves an audit log extraction schema."
* **Verification Method**: Trigger rapid curl scripts to verify the application blocks spam with an HTTP 429 status code and check that submitting an appeal correctly flags the audit log array.
