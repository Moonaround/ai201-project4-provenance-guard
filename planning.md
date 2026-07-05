# ## Architecture & System Specification: Provenance Guard

## 1. ## Architecture Section
When a platform client sends content to Provenance Guard, it follows a structured lifecycle across our isolated backend components. The submission flow validates user inputs against an IP rate limiter before passing text parallelly to a multi-signal extraction pipeline, mathematical weights combine these signals into an ensemble score, and a mapping layer formats plain-language strings before filing data inside an immutable SQLite audit history. The appeal flow reads existing submission states, switches operational status tracking parameters to an under-review flag, and writes content snapshots directly to the log table to await human evaluation.

### Flow Architecture Diagrams
```text
[SUBMISSION FLOW]
POST /submit 
       |
       v
 [Rate Limiter] ---> (Hits Limit) ---> Returns HTTP 429 Error
       | (Under Limit)
       v
 [Three-Signal Ensemble Pipeline]
       |---> Signal 1: Groq LLM Classifier  ---> Output: Float (0.0 to 1.0) ---\
       |---> Signal 2: Length Variance Heuristic -> Output: Float (0.0 to 1.0) ---> [Scoring Engine]

       |---> Signal 3: Type-Token Ratio (TTR)  ---> Output: Float (0.0 to 1.0) ---/        |
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
POST /appeal
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
Our backend utilizes a multi-signal ensemble framework consisting of three completely distinct analytical vectors:
1. **Signal 1: Groq LLM Semantic Classifier**
   * *What it measures*: Phrase structural predictability, semantic smoothing, and token distribution transition density using the `llama-3.3-70b-versatile` model.
   * *Output profile*: A raw float between `0.0` (Human) and `1.0` (AI).
2. **Signal 2: Stylometric Sentence Length Variance**
   * *What it measures*: Pure-Python algorithmic metric tracking the mathematical variance (spread) of word counts per sentence across the body text.
   * *Output profile*: A raw float between `0.0` (Human) and `1.0` (AI).
3. **Signal 3: Lexical Type-Token Ratio (TTR)**
   * *What it measures*: Measures unique word variety density. AI relies on repetitive, high-probability tokens; humans use wider vocabularies.
   * *Output profile*: A raw float between `0.0` (Human) and `1.0` (AI).

#### Signal Combination & Conflict Resolution Formula
The outputs are combined using a weighted average. If individual sub-signals disagree, the math aggregates them down smoothly, allowing borderline content to land safely in the "Uncertain" category:
$$\text{Final Combined Score} = (\text{Groq Score} \times 0.50) + (\text{Variance Score} \times 0.25) + (\text{TTR Score} \times 0.25)$$

### B. Uncertainty Representation & Thresholds
A score of `0.60` represents an uncertain borderline classification. Our system uses asymmetric guard rails to protect human creators from aggressive false positives:
* **0.00 to 0.40 (High-Confidence Human)**: Text matches biological writing parameters with high variance.
* **0.41 to 0.74 (Uncertain / Mixed Provenance)**: Flagged for borderline structural metrics; routes to a protective warning label rather than an outright automated penalty.
* **0.75 to 1.00 (High-Confidence AI)**: Overwhelming markers match generative patterns across signals.

### C. Transparency Label Design
The system uses the following plain-language text variants:
* **High-Confidence Human Variant (`0.00 - 0.40`)**: `"Verified Original: Our system confirms this work matches natural human writing patterns and structural variety."`
* **Uncertain / Mixed Variant (`0.41 - 0.74`)**: `"Mixed Structural Patterns: This text exhibits a combination of predictable phrasing and unique structures. It may contain co-authored or heavily edited content."`
* **High-Confidence AI Variant (`0.75 - 1.00`)**: `"AI-Generated Patterns: Analysis indicates highly uniform sentence structures and vocabulary choices typical of automated writing tools."`
* **Provenance Certified Variant (Premium Badge Overwrite)**: `"🛡️ PROVENANCE CERTIFIED: This piece has been verified through biometric signature identity matching. Absolute Human Authorship Guaranteed."`

### D. Appeals Workflow
* **Who can appeal**: Registered creators who have active content entries saved in our backend database.
* **Information provided**: A text string field containing the creator's justification statement (`creator_reasoning`).
* **System Operations**: The backend checks if the item exists, mutates the submission state column flag to `under_review`, and injects an un-mutable payload block containing their arguments directly into the audit history log.
* **Moderator Queue View**: Human moderators see a table highlighting all records marked `under_review`, presenting the creator's defense text side-by-side with the historical signal metrics and original timestamps.

### E. Anticipated Edge Cases & Structural Failures
1. **Edge Case 1: Legal Disclaimers & Technical Privacy Documents**
   * *Failure Mechanism*: Legal writing demands strict repetitive phrasing and long uniform layouts to ensure compliance. The pipeline will misclassify this formal human writing as machine-generated text due to low variance.
2. **Edge Case 2: Minimalist Free-Verse Creative Poetry**
   * *Failure Mechanism*: Human poets writing avant-garde verse often use deliberately flat, short, three-word sentences across lines. Because sentence length variance drops toward zero, our stylometric heuristic will generate an artificial high score.

---

## 3. Stretch Features Design Specification

### A. Ensemble Detection (+1pt Extra Credit)
Incorporates 3 distinct detection signals (Groq, Length Variance, TTR Score) combined with the 50/25/25 weighting model outlined in Section 2A.

### B. Provenance Certificate (+1pt Extra Credit)
Creators can call `POST /certify` passing a biometric verification token. Validating this token shifts the entry into a permanent certified state, replacing the standard text label with our premium security shield.

### C. Analytics Dashboard (+1pt Extra Credit)
Exposes the `GET /analytics` monitoring channel calculating three operational statistics from SQLite: `total_submissions_monitored`, `ai_verdict_ratio`, and `active_appeal_rate`.

### D. Multi-Modal Support (+1pt Extra Credit)
Allows the system to accept non-text content structures (`content_type: "image_metadata"`). If valid EXIF parameters like `camera_model` are supplied in the request metadata, the engine reduces structural certainty scores by `0.30` to reward genuine physical device capture properties.

---

## 4. ## AI Tool Plan Section

### Milestone 3 (Submission Endpoint & First Signal)
* **Spec Sections Provided**: Section 1 (Diagrams) + Section 2A (Signal 1 Profile).
* **AI Request Prompt**: "Generate a synchronous Flask app skeleton with an initial placeholder database schema alongside a Python function that uses the groq client library to classify input text syntax via a JSON output constraint format."
* **Verification Method**: Pass a hardcoded mock context payload string through the runtime and inspect that a JSON dictionary is returned with the correct status types.

### Milestone 4 (Second Signal & Confidence Scoring)
* **Spec Sections Provided**: Section 1 (Diagrams) + Section 2A (Formula) + Section 2B (Threshold Layout).
* **AI Request Prompt**: "Write a pure-Python string processing method that tokenizes sentences by punctuation markers, measures their word length variances, scales the outputs between 0.0 and 1.0, and executes our weighted fusion formula."
* **Verification Method**: Pass an ultra-repetitive block of text versus a variable narrative passage to verify that the math formula yields distinct numerical changes.

### Milestone 5 (Production Layer & Stretch Features)
* **Spec Sections Provided**: Section 2C + Section 2D + Section 3 (All Stretch Features).
* **AI Request Prompt**: "Build a Flask decorator routing system that implements rate limiting, maps our conditional boundary scales onto verbatim text string labels, handles SQLite under_review mutations, and implements /certify and /analytics endpoints."
* **Verification Method**: Trigger rapid curl scripts to verify the rate limiter, verify all stretch endpoints return valid JSON, and confirm that an appeal updates status correctly.
