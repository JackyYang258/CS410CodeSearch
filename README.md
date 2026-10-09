# CS410CodeSearch

Retrieval-Only Evaluation of Code File Localization for Coding Agents on SWE-bench Lite

## Description
Coding agents must locate relevant code before they edit it, but this localization step is rarely measured alone. We treat code-file localization as an information retrieval problem. Given only the SWE-bench Lite issue text, the system ranks functions and files at the base commit, and we check whether gold-patch files and functions rank near the top.
We compare BM25, dense embeddings, and a hybrid, plus an identifier-regex baseline that approximates agent grep. Repositories are chunked at function and class level with AST parsing. We evaluate 40–60 instances from 3–5 small repositories with File Recall@k, Function Recall@k, and MRR.
The evaluation runs locally with no LLM or agent calls. Scope is retrieval quality only, not patch generation or agent success rate.
Deliverables: an open-source retrieval toolkit with a CLI, documentation, and a usage tutorial.

Keywords: code-search, information-retrieval, coding-agents, fault-localization


## Main deliverables
### A: Data and labels

Select 3–5 small repositories and 40–60 instances; check out base commits; extract gold files and functions

Instance list, gold-label file, loading script

### B: Indexing

AST chunking; metadata; BM25 and embedding indexes; hybrid fusion

Indexing module, index cache

### C: Retrieval methods

Identifier-regex baseline; parameter tuning; query preprocessing

Retrieval modules, parameter settings

### D: Evaluation and tooling

Recall@k, MRR; result tables and plots; failure analysis; CLI

Evaluation script, tables, CLI


### TODO List
Phase 1: Data preparation (A, by Oct 25)

[ ] Select 3–5 small Python repositories. Method: filter by file count, LOC, and instance count.

[ ] Select 40–60 instances. Method: stratified sampling by repository; drop instances whose gold patch touches only non-Python files.

[ ] Check out each base commit. Method: git worktree add from a cached clone.

[ ] Extract gold files and functions. Method: parse diff hunks; map line ranges to functions via the base-commit AST.

[ ] Write the loading script. Method: JSONL; test that every gold file exists at base commit.

Phase 2: Indexing (B, by Nov 8)

[ ] AST chunking into function and class chunks. Method: Python ast; store path, symbol, line range, docstring.

[ ] Metadata storage. Method: JSONL or SQLite per repository.

[ ] BM25 index. Method: bm25s or rank_bm25; split camelCase and snake_case.

[ ] Embedding index. Method: bge-small-en via sentence-transformers; NumPy or FAISS.

[ ] Index caching keyed by repo and commit hash.

Phase 3: Retrieval methods (C, by Nov 8)

[ ] BM25 retrieval. Method: test keeping vs. stripping code blocks in the query.

[ ] Embedding retrieval. Method: cosine similarity over normalized vectors.

[ ] Hybrid retrieval. Method: Reciprocal Rank Fusion, k=60; weighted fusion as an alternative.

[ ] Identifier-regex baseline. Method: extract backticked names, CamelCase and snake_case tokens, file paths; weight path matches higher.

[ ] Tune BM25 k1 and b, the embedding model, and the RRF constant. Method: grid search on a separate small set; never on the evaluation set.

Phase 4: Evaluation (D, Nov 8 to Nov 22)

[ ] File Recall@k for k = 1, 3, 5, 10. Method: set intersection on file paths.

[ ] Function Recall@k. Method: match file and symbol; fall back to line-range overlap.

[ ] MRR. Method: 1-indexed; 0 when no correct chunk appears.

[ ] Full comparison across methods. Method: one script writing a CSV and plots.

[ ] Statistical check. Method: paired bootstrap over instances, 1,000 resamples.

[ ] Failure analysis. Method: group by cause (no identifiers, non-Python gold, rarely changed files, very large files).

Phase 5: Tooling and docs (B and D, Nov 22 to Dec 3)

[ ] CLI with index, query, eval. Method: argparse or typer.

[ ] Usage README with installation and one end-to-end example.

[ ] Implementation docs in docs/: chunking, indexing, fusion, metric definitions.

[ ] Reproducibility check from a clean clone. Method: fixed seeds, pinned requirements, cached indexes.

Phase 6: Milestones

[ ] Oct 11: proposal submitted (Coordinator).

[ ] Nov 22: progress report with #progress (Coordinator).

[ ] Dec 3: code freeze; documentation drafts done.

[ ] Dec 10 (Reading Day): push code to GitHub (D); submit docs (B, D); upload tutorial video to Illinois Media Space (C, with A); add repo and video links to TextData and tag #report (Coordinator).

### Risks and Mitigations
- Large repositories slow indexing. Limit to small repos and cache every index.
- CPU embeddings are slow. Use a small local model; use a GPU if available.
- Gold functions are hard to map after patching. Map by file, then line range; if mapping stays noisy, report File Recall as the primary metric.
- Schedule slips. BM25 is the minimum viable product; embedding and hybrid retrieval can be dropped.
- No LLM calls means no generation results. This is intentional; claims stay limited to retrieval quality.
