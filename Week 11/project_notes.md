# Week 11 – Project 3: RAG Financial Intelligence Assistant (GlobalEdge Brokerage)

> Design notes for the project. The final deliverable is in English; these notes can be in Spanish or English.
> Track: **Full Code**. Working notebook: `Ignacio Leites - Week 11 project.ipynb` (currently an identical copy of `AI&DS_Project_3_Full_Code_Notebook.ipynb`).
> Deliverable: **only the exported `.html`**, run top-to-bottom on a clean kernel, no errors, all outputs visible.

---

## 1. Project brief (summary)

**Business problem.** GlobalEdge Brokerage: 400 brokers, ~450 clients each. In the 7:00–9:15 AM prep window, 60–80 news articles + SEC filings are published; a broker reads only ~20–30%. Consequences:
- Recommendations made without today's signals.
- Brokers can't cite sources when challenged → trust and compliance risk.
- Compliance review happens *after* client calls.
- Senior brokers resist tooling that adds friction.

**Objective.** PoC of a natural-language RAG assistant over news + prices + SEC filings that:
- Answers plain-English broker questions with grounded answers.
- **Always surfaces sources** (title, filing reference, publication date).
- Success = brokers expand pre-call coverage within the first week of use.

## 2. Data

All files live in `AI&DS_Project_3_Datasets/` (so `DATA_DIR = Path("AI&DS_Project_3_Datasets")`).

| File | Rows / size | Columns / content |
|---|---|---|
| `global_news.csv` | 760 articles | title, description, content (truncated, ~200 chars + "[+N chars]"), source, url, published_at, query |
| `stock_price_details.csv` | 2,986 rows | Date, ticker, name, Open, High, Low, Close, Volume (23 tickers: stocks, FX, commodities, crypto) |
| `sec_filings_10k.txt` | ~8 MB plain text | 10-K filings (Microsoft, Tesla, Alphabet, Amazon, Apple…) incl. XBRL noise at the start |
| `golden_benchmark_dataset.csv` | 20 rows | question, response, source_hint, supporting_sources, context |

**Discrepancies to keep in mind:**
- The intro mentions `sec_filings_10q.pdf` (126-page 10-Q, Q1 2026); the actual file is `sec_filings_10k.txt` (10-K annual reports). The template text follows the `.txt` / 10-K version → we use what's in the folder. **Confirmed on the course forum (2026-11-15):** the `.txt` is intentional; follow the project instructions with it.
- Final test case 1 asks about "latest **10-Q** filings" → the system only has 10-Ks; worth commenting on in the observations (good test of grounding / not hallucinating).
- News `content` is truncated → `title` + `description` carry most of the signal.
- The SEC text has inline XBRL/HTML-entity noise (`&#160;`, `&#8220;`, URLs) → consider light cleaning before chunking.

## 3. Rubric (60 pts)

| Criterion | Pts | Notebook section |
|---|---|---|
| Data preparation for RAG (load, splitter + justification, embedding model + justification, vector store, observations) | 10 | 1.1–1.3 |
| QA with RAG (LLM + params justified, system prompt, retriever, answer test cases, observations) | 10 | 1.4, 4.x |
| Evaluating RAG (eval LLM justified, eval params, evaluate 5 test cases, observations) | 8 | 1.5–1.7, 2.x |
| Tuned RAG (3 configs of chunk size/overlap/k/temperature/top_p/max_tokens, justified; answer + evaluate; observations) | 20 | 3.1–3.7 |
| Actionable insights & recommendations | 4 | Conclusion |
| Overall quality (structure, flow, commented code, conclusion) | 8 | whole notebook |

## 4. Notebook template structure

- Setup: installs (chromadb, langchain-*, deepeval 4.2.2…), imports, **Environment Setup** (paths, API key / base URL, model names).
- **Section 1 – Baseline RAG**: 1.1 load CSVs (`CSVLoader`) · 1.2 load + split SEC text · 1.3 Chroma vector store · 1.4 retrieve + generate · 1.5 load golden benchmark · 1.6 GEval metrics (Relevance, Faithfulness, Completeness — criteria + thresholds to fill) · 1.7 evaluate baseline.
- **Section 2 – DeepEval prompt optimization (GEPA)**: 2.1 prompt template + model callback (pre-written, temps to fill) · 2.2 GEPA (retry env vars, iterations, pareto_size, minibatch_size, seed to fill) · 2.3 evaluate optimized prompt · 2.4 compare vs baseline.
- **Section 3 – RAG config tuning**: 3.1 define configs · 3.2 runtime answer fn · 3.3 evaluate one config · 3.4 run all · 3.5 compare · 3.6 pick best · 3.7 save results.
- **Section 4 – Final inference**: 4.1 helper · 4.2 five final test cases (given) · 4.3 run them.
- **Conclusion**: Actionable insights + Recommendations.

Variables the pre-written cells expect us to define: `EVAL_MODEL`, `ANSWER_MODEL`, `BASELINE_SYSTEM_PROMPT`, `BASELINE_USER_PROMPT` (with `{question}` / `{context}` placeholders), `DEEPEVAL_METRICS`, plus whatever GEPA needs for goldens.

Reference: `AI&DS_Project_3_Low_Code_Notebook.ipynb` has the reference implementation with blanks (helper functions, `format_docs`, `retrieve_docs`, etc.) → mirror its approach, written from scratch.

## 5. Design decisions

_(to be filled as we go)_

- **Environment:** run **locally**, conda env **`mit-env`** (Python 3.11) — already has the exact pinned versions (chromadb 1.5.9, langchain-community 0.4.1, deepeval 4.2.2, pandas 3.0.3, ipykernel). Keep the `%pip install` cell as in the template.
- **Credentials:** read from `/Users/nacho/MIT/MIT/config.json` (relative to the notebook: `../config.json`) instead of `getpass`. The file uses keys `OPENAI_API_KEY` and `OPENAI_API_BASE` (Great Learning proxy) → the notebook expects `OPENAI_BASE_URL`, so map `OPENAI_API_BASE → OPENAI_BASE_URL`. **Never print the key.**
- **Course API rules (Great Learning):**
  - Only `gpt-4o` and `gpt-4o-mini` allowed (other names → Error 400).
  - API base must be exactly `https://aibe.mygreatlearning.com/openai/v1` (otherwise → Error 401). `config.json` matches ✔.
  - Key has a **fixed $20 credit for the whole program** (MLS exercises + Final Project), no top-ups. Current balance believed ~10 USD.
  - Smoke test 2026-11-15 (tiny calls): `text-embedding-3-small` ✔ (1536 dims), `gpt-4o-mini` ✔, `gpt-4o` ✔ through the proxy.
- **Budget: ~10 USD left, shared with future coursework → be austere.** Rules:
  - **Decision (user): use `gpt-4o` wherever possible** → `ANSWER_MODEL = "gpt-4o"` and `EVAL_MODEL = "gpt-4o"`. Fallback: `ANSWER_MODEL = "gpt-4o-mini"` if spend gets tight (eval stays on gpt-4o).
  - Justification: gpt-4o is the stronger of the two allowed models → better financial reasoning, numeric comparison and instruction following (citations, "no 10-Q available") for answers, and a more reliable judge for GEval.
  - Cost impact: gpt-4o is ~17x the price of gpt-4o-mini → rough full run ≈ $3–5 (vs $1–3). **Add a token/cost tracker** in the notebook and keep GEPA + eval sets small.
  - Small GEPA (few iterations, small minibatch/pareto).
  - Evaluate configs only on the **test split** of the golden set (template already splits 20 goldens into train/test).
  - **Don't re-embed on every run:** persist Chroma and reuse the collection if it already exists (`Chroma.from_documents` on an existing persist dir appends duplicates). One collection per chunking config.
  - Cache expensive outputs (eval results, GEPA prompt) in `ARTIFACT_DIR` during development; one final clean top-to-bottom run for the HTML.
  - Rough estimate: embeddings of the 8 MB SEC file ≈ 2M tokens ≈ $0.04 per chunking config; full notebook run with gpt-4o judge ≈ $1–3. → room for ~2–3 full runs.
- Answer model / eval model: see budget above (template options: `gpt-4o-mini` / `gpt-4o`).
- Embedding model: `text-embedding-3-small` (template default) — justify: cost, 1536 dims, good on financial text.
- Splitter: `RecursiveCharacterTextSplitter` for SEC text; CSV rows kept as one document each.
- Chunk size / overlap: TBD.
- k (retrieved chunks): TBD.
- Metric thresholds: TBD.
- GEPA params: TBD (keep small to control cost/time).
- 3 tuning configs: TBD.

## 6. Open questions / to discuss

- ~~Where to run / credentials~~ → local, `mit-env`, `../config.json` (see §5).
- ~~Budget~~ → ~10 USD, austere (see §5).
- ~~Eval model~~ → `gpt-4o` (see §5).
- Clean the SEC XBRL noise or index raw?
- Should price rows be aggregated (e.g., per ticker per week) to make trend questions answerable, or keep one row per document as in the template?

## 7. Log

- **2026-11-15** – Project kickoff. Read the brief, inspected datasets and both template notebooks, created these notes.
- **2026-11-15** – Checked `ey-sec29172-251us-12-04-2025.pdf`: it's EY's "SEC Financial Reporting Series – Form 10-Q" guide (80 pages, Dec 2025), a how-to for *preparing* 10-Qs. No company financials (no Apple/Alphabet/Tesla data) → **not** a substitute for `sec_filings_10q.pdf`. Not used; we stick with `sec_filings_10k.txt`.
- **2026-11-15** – Forum answer: the `.txt` 10-K file is intentional (10-Ks are often distributed as plain text). Discrepancy closed → `FILINGS_TXT = DATA_DIR / "sec_filings_10k.txt"`, loaded with `TextLoader` as in the template.
  - **To do in 4.3 observations (test case 1):** the question asks for "latest 10-Q filings" but the corpus only has 10-Ks → the expected/good behavior is to answer with the 10-K figures and explicitly state that no 10-Q is available (no invented quarterly numbers). Comment on whether the model does this.
- **2026-11-15** – Run locally in conda env `mit-env`; credentials from `../config.json`; budget ~10 USD → austerity rules in §5.
- **2026-11-15** – Course API guidelines added to §5 ($20 fixed credit for the whole program; only gpt-4o / gpt-4o-mini; fixed API base). Smoke-tested the proxy: embeddings + both chat models OK. Decided `ANSWER_MODEL = gpt-4o-mini`, `EVAL_MODEL = gpt-4o`.
- **2026-11-15** – Changed: use gpt-4o wherever possible → `ANSWER_MODEL = EVAL_MODEL = "gpt-4o"`, gpt-4o-mini as fallback for answers if budget gets tight.
