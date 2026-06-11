# Agent Context

This file captures the project context loaded during the session on 2026-06-10.
Primary sources were CodeGraph MCP and Understand-Anything artifacts already present in the repo.
This is a working context file for future sessions, not a product README.

## Snapshot

- Repo: `retrieval-backend-with-rag`
- Current git HEAD at analysis time: `ebc3dc899f3da67c7de7b010190e1ac513d4e74d`
- Main runtime: Python 3.12
- API framework: Flask
- Project purpose: Vietnamese RAG backend and evaluation toolkit for product Q&A plus chitchat routing

## Graph Status

### CodeGraph

- `.codegraph/` exists at repo root.
- CodeGraph status during analysis:
  - Files indexed: 36
  - Nodes: 433
  - Edges: 666
  - Backend: `node:sqlite`
- CodeGraph was healthy and usable. No re-init was needed.

### Understand-Anything

- `.understand-anything/knowledge-graph.json` exists.
- `.understand-anything/meta.json` exists.
- `.understand-anything/config.json` exists.
- `.understand-anything/domain-graph.json` was not present.
- Important mismatch:
  - `meta.json.gitCommitHash = non-git-worktree`
  - `knowledge-graph.project.gitCommitHash = non-git-worktree`
  - actual `git rev-parse HEAD = ebc3dc899f3da67c7de7b010190e1ac513d4e74d`
- Conclusion:
  - Use Understand-Anything for high-level onboarding only.
  - Use CodeGraph as the main source for current architecture, symbols, call flow, and impact analysis.

## Tech Stack

Based on `pyproject.toml`, `requirements.txt`, CodeGraph, and the current file tree:

- Language: Python
- Package manager: `uv`
- API: `flask`, `flask-cors`
- Validation/config types: `pydantic`
- Vector stores:
  - `chromadb`
  - `qdrant-client`
  - `pymongo` for MongoDB Atlas vector search
- Embeddings:
  - `sentence-transformers`
  - local embedding wrappers in `embeddings/`
  - `fastembed`
- LLM providers / runtimes:
  - `openai`
  - `google-generativeai`
  - `mistralai`
  - local `ollama`
  - local `vllm`
  - local `onnxruntime`
  - local `transformers` Hugging Face inference
- Evaluation:
  - `ragas`
  - `sacrebleu`
- Other notable deps:
  - `langchain`
  - `httpx`
  - `vertexai`
  - `google-cloud-aiplatform`
  - `accelerate`
  - `gdown`

## High-Level Architecture

Understand-Anything identified these layers, and CodeGraph matched them well enough structurally:

1. Project Setup
   - `pyproject.toml`
   - `requirements.txt`
   - `.python-version`
   - declared `README.md`, but the actual file in the repo is `README_old.md`

2. Interface Layer
   - `index.html`
   - appears minimal and likely only for light browser checks

3. API & Orchestration
   - `serve.py`
   - `semantic_router/`

4. RAG Pipeline
   - `rag/core.py`
   - `re_rank/core.py`
   - `reflection/core.py`

5. Model Integration Layer
   - `llms/`
   - `embeddings/`

6. Data Assets & Ingestion
   - `insert_data/build_chromadb.py`
   - `data/`
   - `hoanghamobile.csv`

7. Evaluation & Benchmarking
   - `benchmark_inference_time_llms.py`
   - `benchmark_inference_time_pipeline.py`
   - `test/unitTest/`
   - `test/integrationTest/`

## File and Module Inventory

CodeGraph indexed the following main Python areas:

- `serve.py`
- `embeddings/`
  - `base.py`
  - `fastEmbed.py`
  - `google.py`
  - `mistral.py`
  - `openai.py`
  - `sentenceTransformer.py`
- `insert_data/`
  - `build_chromadb.py`
- `llms/`
  - `llms.py`
  - `localLlms.py`
  - `onlinesLlms.py`
  - `onnx.py`
- `rag/`
  - `core.py`
- `re_rank/`
  - `core.py`
- `reflection/`
  - `core.py`
- `semantic_router/`
  - `route.py`
  - `router.py`
  - `samples.py`
- `test/unitTest/`
  - `test_api_response.py`
  - `test_chromadb_vector_search.py`
  - `test_hit@k_mongodb.py`
  - `test_hit@k_qdrant.py`
  - `test_huggingface_backend.py`
  - `test_onnx_backend.py`
  - `test_reflection.py`
  - `test_rerank_ncdg.py`
- `test/integrationTest/llm-answer/`
  - `test_bleu.py`
  - `test_rouge.py`

## Entry Points

### Main HTTP Entry Point

- `serve.py` is the backend entry point.
- `main(args)` wires up routing, LLM backend, reflection, vector store, reranker, and Flask.
- Flask route:
  - `POST /api/search`
- Server startup:
  - `app.run(host='0.0.0.0', port=5002, debug=True)`

### CLI Arguments in `serve.py`

- `--mode`: `online` or `offline`
- `--model_name`
- `--model_engine`
- `--model_version`
- `--db`: `qdrant`, `mongodb`, `chromadb`
- `--embedding_model`
- `--reranker`

### Secondary Entry Points

- `benchmark_inference_time_llms.py`
- `benchmark_inference_time_pipeline.py`
- tests under `test/`

## Core Runtime Flow

The current request flow inferred from CodeGraph is:

1. Client sends a JSON array of chat messages to `POST /api/search`.
2. `handle_query()` reads the body with `request.get_json()`.
3. The full conversation is passed to `Reflection`.
4. `Reflection.__call__()` rewrites or reformulates the latest user request into a standalone Vietnamese query.
5. That rewritten query is routed by `SemanticRouter.guide()`.
6. Two route families exist:
   - `products`
   - `chitchat`
7. If routed to `products`:
   - `rag.vector_search(query)` retrieves candidate documents from the configured vector store
   - passages are passed into `Reranker`
   - ranked passages are concatenated into a sales-assistant prompt
   - the prompt is appended to the message list
   - generation is executed via `rag.generate_content(data)`
8. If routed to `chitchat`:
   - retrieval is skipped
   - generation goes directly through `llm.generate_content(data)`
9. API returns:
   - `{"content": response, "role": "assistant"}`

## Semantic Routing Layer

### Purpose

The semantic router decides whether a user request should go through RAG or go directly to the LLM.

### Main Components

- `semantic_router/router.py`
  - `SemanticRouter`
  - `guide(query)`
- `semantic_router/route.py`
  - route container
- `semantic_router/samples.py`
  - route examples for `products` and `chitchat`

### Routing Logic

- Route sample embeddings are precomputed on initialization.
- Query embedding is compared against route sample embeddings using cosine-similarity-like averaging.
- The highest score wins.

### Important Note

- The routing is done after query reflection, not before.
- This means all requests pay the reflection cost first, even requests that will become simple chitchat.

## Reflection Layer

### Purpose

`reflection/core.py` is a query rewriting layer that converts contextual chat requests into a standalone Vietnamese question before routing and retrieval.

### Current Behavior

- Truncates history if it exceeds `lastItemsConsidereds`
- Concatenates the conversation into a plain text prompt
- Asks the LLM:
  - reformulate the latest question into a standalone Vietnamese question
  - do not answer, only rewrite if needed
- Returns the LLM output as the query used downstream

### Implication

- Reflection is a hard dependency for every request path.
- If the LLM backend is unavailable, even routing can fail.
- Reflection latency is part of end-to-end latency for both `products` and `chitchat`.

## RAG Layer

### Main Class

- `rag/core.py`
  - class `RAG`

### Supported Backends

- `chromadb`
- `mongodb`
- `qdrant`

### Main Responsibilities

- initialize vector store client
- initialize sentence-transformer embeddings
- encode query text into embeddings
- run vector search
- delegate final generation to the configured LLM wrapper

### Current Retrieval Behavior

- `qdrant`
  - searches collection named from `embeddingName.split('/')[-1]`
  - returns `_id`, `combined_information`, `score`
- `mongodb`
  - uses Atlas `$vectorSearch`
  - expects index named `vector_index`
  - projects product fields and score
- `chromadb`
  - queries local persistent collection
  - converts distance into `1 - distance` similarity

## Reranking Layer

### Main Class

- `re_rank/core.py`
  - class `Reranker`

### Role in Pipeline

- Receives the rewritten query and retrieved passages
- Produces scores and ranked passages
- Ranked passages are then injected into the final answer prompt

### Position in Flow

- retrieval first
- reranking second
- answer generation third

## LLM Integration Layer

### Facade

- `llms/llms.py`
  - class `LLMs`
  - wraps either local or online implementation

### Online Backends

- `llms/onlinesLlms.py`
  - supports:
    - `gemini`
    - `openai`
    - `together`

Behavior:

- Gemini:
  - uses `google.generativeai`
- OpenAI:
  - uses `openai.OpenAI`
- Together:
  - uses direct HTTP POST to `/v1/chat/completions`

### Local / Offline Backends

- `llms/localLlms.py`
  - supports:
    - `ollama`
    - `vllm`
    - `onnx`
    - `huggingface`

Behavior:

- `ollama`
  - checks server connectivity
  - auto-pulls model if missing
- `vllm`
  - checks `/v1/models`
  - infers `max_tokens`
- `huggingface`
  - loads tokenizer and model locally via `transformers`
- `onnx`
  - delegates to `ONNXModel`

### Output Sanitization

- Both local and online wrappers include logic to strip `<think>...</think>` blocks from generated text in some paths.

## Embedding Layer

### Main Observations

- `SentenceTransformerEmbedding` is the embedding class actively used in `serve.py` and `rag/core.py`.
- The repo contains multiple embedding wrappers in `embeddings/`.
- Semantic routing and retrieval use embedding-based similarity, but in different contexts:
  - semantic routing for route choice
  - vector search for document retrieval

## Data Ingestion

### Main Utility

- `insert_data/build_chromadb.py`
  - `load_csv_to_chromadb(csv_path, persist_dir="./chroma_db", model_name="Alibaba-NLP/gte-multilingual-base")`

### Current Ingestion Flow

1. Read CSV
2. Build `combined_information` by concatenating all columns
3. Create embeddings for each row
4. Open local Chroma persistent client
5. Create or get collection named from model name
6. Add ids, documents, embeddings, and product metadata

### Automatic Ingestion from `serve.py`

- When `--db chromadb` is selected:
  - the server checks whether the target collection already exists
  - if not, it scans `data/` for CSV files
  - it ingests all CSVs found before serving requests

### Main Data Asset

- `hoanghamobile.csv`
- likely product catalog used for Vietnamese phone-store Q&A

## Environment and External Dependencies

The project expects environment variables for the selected backend.

### Online LLMs

- `GEMINI_API_KEY`
- `OPENAI_API_KEY`
- `TOGETHER_API_KEY`
- `TOGETHER_BASE_URL`

### Offline Runtime Servers

- `OLLAMA_BASE_URL`
- `VLLM_BASE_URL`

### Vector Stores

- MongoDB:
  - `MONGODB_URI`
  - `MONGODB_NAME`
  - `MONGODB_COLLECTION`
- Qdrant:
  - `QDRANT_API`
  - `QDRANT_URL`

## Tests and Benchmarks

### Unit Test Coverage Areas

- API response behavior
- Chroma vector search
- MongoDB hit@k
- Qdrant hit@k
- Hugging Face backend
- ONNX backend
- reflection behavior
- reranker NDCG

### Integration / Evaluation Areas

- BLEU evaluation
- ROUGE evaluation
- CSV-based answer datasets

### Benchmark Scripts

- `benchmark_inference_time_llms.py`
  - likely compares LLM inference times across providers/backends
- `benchmark_inference_time_pipeline.py`
  - likely benchmarks the full retrieval + rerank + generation path

## Documentation State

- `pyproject.toml` declares `readme = "README.md"`
- actual file in repo is `README_old.md`
- This mismatch should be treated as a documentation/packaging inconsistency.

## Confirmed Risks and Code Smells

These were identified directly from the graph-exposed source, not guessed.

### 1. `_collection_exists` is used like a property instead of a method

In `rag/core.py`, both of these use `self._collection_exists` instead of `self._collection_exists()`:

- in Chroma initialization
- in Qdrant search path

This is a real logic risk because a bound method object is truthy.
Expected consequences:

- collection existence checks may not behave as intended
- branches may always be treated as true
- initialization/search behavior can silently drift from intent

### 2. Possible typo in ingestion column handling

In `insert_data/build_chromadb.py`:

- it checks for `'combined_infomation'`
- then drops `'combined_information'`

This looks inconsistent and is likely a bug or stale typo.
It can break or distort ingestion depending on the CSV columns present.

### 3. Reflection is globally on the hot path

- Every request appears to go through `Reflection` before routing.
- This increases latency and couples routing to LLM availability.

### 4. `process_query()` is dead or unused

- `serve.py` defines `process_query(query)` returning `query.lower()`
- no active path found in the analyzed flow used it

### 5. Startup mode is development-oriented

- Flask is launched with `debug=True`
- acceptable for local work, but operationally risky for production

### 6. README reference is stale

- manifest points to `README.md`
- repo contains `README_old.md`

## Understand-Anything Tour Summary

The prebuilt knowledge graph suggested this onboarding order:

1. Project overview
   - manifests and docs
2. API entry and routing
   - `serve.py`
   - semantic router
3. RAG processing path
   - `rag/core.py`
   - `re_rank/core.py`
   - `reflection/core.py`
4. Model and embedding backends
   - `llms/`
   - embedding adapters
5. Data ingestion and assets
   - `insert_data/build_chromadb.py`
   - CSV assets
6. Evaluation and benchmarks
   - benchmark scripts
   - integration/unit tests

This tour is still useful as a reading order, but not as proof of current commit accuracy.

## Recommended Future-Session Bootstrap

When starting a new session on this repo:

1. Read `AGENTS.md`
2. Read this `AGENT.md`
3. Verify current `git rev-parse HEAD`
4. Check `mcp__codegraph.codegraph_status`
5. Use CodeGraph for:
   - symbol lookup
   - call flow
   - architecture tracing
   - impact analysis
6. Treat `.understand-anything/knowledge-graph.json` as high-level background unless its commit hash matches current HEAD

## Practical Mental Model

The backend is a small Flask-based orchestration service for product Q&A in Vietnamese.
Its core design is:

- rewrite question from chat context
- semantically classify the query
- if product-related, retrieve product context, rerank it, and answer with grounded prompt
- otherwise answer directly with the chosen LLM backend

The codebase is compact enough that `serve.py` is the control tower, while `rag/`, `reflection/`, `re_rank/`, `llms/`, and `semantic_router/` hold the actual behavior.

## Scope of This File

This file records what was actually analyzed from:

- CodeGraph status
- CodeGraph context/search/node/explore results
- Understand-Anything metadata and knowledge graph high-level sections
- `pyproject.toml`
- `requirements.txt`
- `README_old.md`

It is not a full manual audit of every line in the repository.
If a future task requires exact behavior of a specific symbol, prefer CodeGraph first before manually reading files.
