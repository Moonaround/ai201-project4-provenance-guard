# 🛡️ Provenance Guard — Production Verification Record

## 1. Multi-Signal Pipeline & Ensemble Scoring Architecture
Our platform rejects simplistic, binary classification in favor of a balanced, multi-signal ensemble engine. The pipeline processes raw incoming text through two completely decoupled analytical components to eliminate blind spots:

*   **Signal 1: Groq LLM Contextual Classifier (Weight: 0.60)**: Uses the `llama-3.3-70b-versatile` model to evaluate semantic structural transitions, predictability, and holistic tonal uniformity.
    *   *What it captures*: Synthesized phrasing and machine structural patterns.
    *   *What it misses*: Short texts lacking semantic weight or highly formal professional human writing.
*   **Signal 2: Sentence Length Variance Heuristic (Weight: 0.40)**: A pure-Python component that extracts individual sentences and evaluates the mathematical variance of their word counts to detect structural rhythm ("burstiness").
    *   *What it captures*: Rhythmic complexity. Human authors naturally alternate sentence sizes dynamically, whereas generative tools favor narrow, highly uniform length distributions.
    *   *What it misses*: Short compositions or minimalist structured human poetry.

### Calibrated Fused Confidence Scoring
The individual normalized metrics (0.0 for pure human traits, 1.0 for machine traits) are combined using a weighted average:
\[\text{Final Confidence Score} = (\text{Groq Score} \times 0.60) + (\text{Variance Score} \times 0.40)\]

---

## 2. Evidence of Meaningful Score Variation
Our scoring engine produces clear, meaningful numeric variation across contrasting test examples rather than standard constants or sudden binary flips:

### High-Confidence Case (AI-Generated Profile)
*   **Input Text**: *"Artificial intelligence represents a transformative paradigm shift in modern society. It is important to note that while the benefits of AI are numerous, it is equally essential to consider the ethical implications. Furthermore, stakeholders across various sectors must collaborate to ensure responsible deployment."*
*   **Signal Metrics**: Groq Score = `0.90`, Sentence Variance Score = `0.85`
*   **Final Fused Score**: **`0.88`**
*   **Resulting Category / Verdict**: `likely_ai`

### Lower-Confidence Case (Borderline / Uncertain Profile)
*   **Input Text**: *"I've been thinking a lot about remote work lately. There are genuine tradeoffs. Flexibility and no commute on one side, isolation and blurred work-life boundaries on the other. Studies show productivity varies widely by individual and role type."*
*   **Signal Metrics**: Groq Score = `0.40`, Sentence Variance Score = `0.85`
*   **Final Fused Score**: **`0.58`**
*   **Resulting Category / Verdict**: `uncertain`

---

## 3. Plain-Language Transparency UX Table
To protect creators and maximize user clarity, our backend maps confidence boundaries to plain-language transparency labels that completely omit technical jargon like "logits", "classifier output", or "probability densities":

| Category | Score Range | Verbatim UX Transparency Label Text |
| :--- | :--- | :--- |
| **HUMAN** | `0.00 - 0.40` | "Verified Original: Our system confirms this work matches natural human writing patterns and structural variety." |
| **UNCERTAIN** | `0.41 - 0.74` | "Mixed Structural Patterns: This text exhibits a combination of predictable phrasing and unique structures. It may contain co-authored or heavily edited content." |
| **AI** | `0.75 - 1.00` | "AI-Generated Patterns: Analysis indicates highly uniform sentence structures and vocabulary choices typical of automated writing tools." |

---

## 4. Production Rate Limiting Logic & Evidence
We enforce a strict boundary configuration of **10 requests per minute per IP address** via `Flask-Limiter`.
*   **Usage Reasoning**: Organic human authors typing stories, poetry, or saving platform drafts submit updates every few minutes. A threshold of 10 requests per minute handles natural creative workflows effortlessly while instantly blocking automated scraping loops, bot floods, or malicious spam scripts.
*   **Evidence of Rate Limiting (HTTP 429 Error Codes)**:
    When executing a rapid script loop sending 12 sequential payloads to the `/submit` endpoint, our safety layer catches the abuse, printing `201 Created` for the first 10 and shifting to explicit `429 Too Many Requests` status codes immediately after:
    ```text
    201
    201
    201
    201
    201
    201
    201
    201
    201
    201
    429
    429
    ```

---

## 5. Known System Limitations
*   **Target Misclassification Failure**: The system systematically misclassifies **Legal Terms of Service Agreements, Privacy Policies, and Standard Disclaimers** written entirely by human attorneys.
*   **Explanation tied to signals**: Legal drafting explicitly demands strict repetitive phrasing (low lexical diversity) and highly uniform layout constraints (low sentence variance) to maintain absolute regulatory compliance. Because this matches the low-variance profile of machine-generated text, it triggers high false-positive AI labels.

---

## 6. Specification Design Reflection
*   **Guided by Spec**: The API surface contract written in `planning.md` ensured that our JSON payloads stayed consistent across all endpoints. When building out the `/appeal` endpoint later, we already knew exactly which schema properties were required.
*   **Divergence from Spec**: The original `planning.md` intended to call an external heavyweight Python text complexity module to assess readability indexes. 
*   **Reason for Divergence**: During deployment, the module caused environment version lock issues. We pivoted to pure-Python regex string variance extraction, maintaining lightning-fast processing speed and lightweight environment stability without external package complexity.

---

## 7. AI Tools Log
*   **Instance 1 (FastAPI to Flask Conversion)**: Directed an AI helper to refactor async routing paths into standard synchronous Flask app decorators.
    *   *Student Revision/Override*: The AI generated broken database connection calls inside the decorators. We overrode the connection handling to open cleanly using transactional context structures (`with sqlite3.connect...`).
*   **Instance 2 (Variance Scaling Logic)**: Directed an AI helper to normalize variable word metrics onto a standard decimal base line.
    *   *Student Revision/Override*: The AI code inverted the metrics, labeling human writing as automated text. We manually inverted the math outputs to ensure accuracy.
