# Lab 08 - RAG and Enterprise Context Pipelines

**Course:** Advanced Software Development with Agentic AI (ASD)  
**Theme:** RAG and Enterprise Context Pipelines  
**Primary IDE:** VS Code  
**AI Runtime:** Ollama  
**Primary Local Model (Open Source):** Qwen 2.5 0.5B  
**Secondary Local Review Model (Open Source):** Llama 3.1 8B  
**Duration:** 120 Minutes  

## 1. Overview

<details>
<summary>Goal</summary>

Extend the Student Enrolment System with RAG pipeline and integrate with agentic_loop (now supporting MCP and RAG modes) for automated validation.

Students will:

* Build RAG server with corpus refresh, context retrieval, and grounded answering
* Add RAG mode to agentic_loop (alongside existing MCP mode)
* Execute RAG validation through agentic workflow
* Capture and review retrieval metrics and answer quality

</details>

<details>
<summary>Key RAG Concepts</summary>

* **Data/files → Corpus**: all information RAG can search.
* **Chunks**: small pieces of the corpus.
* **Vectors**: numerical representations of the meaning of each chunk.
* **ChromaDB**: stores/indexes vectors and finds similar chunks.
* **k**: number of chunks retrieved. **k=5 → top 5 chunks**.
* **Citations**: identify the original sources/chunks used in the answer.
* **Tool Contract**: defines what a tool does, its inputs, and outputs.
* **P@5 (Precision at 5)**: of the top 5 retrieved chunks, how many were relevant.
* **R@5 (Recall at 5)**: of all relevant information, how much was found within the top 5.

**RAG Workflow:**

`Data/Files → Corpus → Chunks → Vectors → ChromaDB → Retrieve Top-k Chunks → LLM → Answer + Citations`

**Example:**

`students.pdf → Corpus → Chunk 12: "John is enrolled in ASD101" → Vector → ChromaDB → Retrieve Top 5 → LLM → "John is enrolled in ASD101" + Citation: students.pdf, Chunk 12`

**Vector = helps find the information.**
**Citation = tells you where the information came from.**

</details>

<details>
<summary>RAG Integration Pattern</summary>

REFRESH → RETRIEVE → ANSWER → VALIDATE → REVIEW → IMPROVE

</details>

<details>
<summary>Scope</summary>

**In scope:**

* Local RAG pipeline (chunk, embed, index, retrieve)
* ChromaDB vector store
* MCP RAG tools integration
* Agentic_loop RAG mode automation

**Out of scope:**

* Cloud services or commercial APIs
* Production deployment

**Important Architecture Note:**

* **RAG server runs locally** (NOT containerized) - similar to MCP server in Lab 07
* RAG HTTP server must be running on port 5003 while Docker services are active
* Containerized services connect to local RAG via `host.docker.internal:5003`
* This allows development/testing without containerizing every component

</details>

<details>
<summary>Expected Results</summary>

By the end of this lab, students should have:

* RAG server with corpus refresh, retrieval, and Q&A tools
* RAG mode integrated into agentic_loop
* RAG collector and pipeline modules
* Automated RAG validation workflow
* Retrieval metrics (P@5, R@5) and answer quality analysis
* Evidence reports (rag-validation-report.md)

</details>

---

## 2. Prerequisites and Configuration

<details>
<summary>Prerequisites</summary>

Complete Labs 01-07.

Required:

* Docker Desktop (for frontend, enrolment-service, database-service only)
* Ollama with qwen2.5:0.5b and llama3.1:8b
* Python virtual environment
* Lab 07 MCP integration complete
* Lab 05 CI evidence (run `workflow_dispatch` if missing)

**Note:** RAG server runs locally (not containerized), similar to MCP server in Lab 07.

</details>

---

## 3. Scenario Setup

<details>
<summary>Student Enrolment System</summary>

Lab 07 delivered MCP tool integration with agentic_loop validation.

Lab 08 adds RAG pipeline:

* RAG server with corpus refresh, retrieval, and Q&A
* agentic_loop validates RAG pipeline automatically
* Retrieval metrics (P@5, R@5) captured
* Evidence captured for review

</details>

<details>
<summary>Business Requirement</summary>

The business requires agents to answer questions using retrieved evidence, not hallucinations.

Requirements:

* Answers must cite retrieved sources
* Confidence scores must reflect retrieval quality
* All retrieval must be auditable

</details>

<details>
<summary>Project Structure</summary>

```text
enrolment-app-open-ai/
│
├── .github/
│   └── workflows/
│       └── lab5-ci.yml
│
├── docker-compose.yml
│
├── agentic_loop/
│   ├── config/
│   │   └── review_config.py           # add rag mode
│   ├── core/
│   ├── collectors/
│   │   ├── mcp_collector.py           # Lab 7
│   │   └── rag_collector.py           # new in Lab 8
│   └── pipelines/
│       ├── mcp_pipeline.py            # Lab 7
│       └── rag_pipeline.py            # new in Lab 8
│
├── frontend-service/
│   ├── Dockerfile
│   ├── css/
│   │   └── styles.css
│   └── templates/
│       ├── index.html
│       └── tabs/
│           ├── normal.html
│           ├── ai-mode.html
│           ├── mcp.html
│           └── rag.html               # new in Lab 8
│
├── enrolment-service/
│   ├── app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── routes/
│   │   ├── mcp_mode.py
│   │   └── rag_mode.py                # new in Lab 8
│   └── services/
│       ├── database_api.py
│       ├── llm_client.py
│       ├── prompt_loader.py
│       └── rag_api.py                 # new in Lab 8
│
├── database-service/
│   ├── app.py
│   ├── init_db.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── data/
│       └── enrolment.db
│
├── mcp-server/
│   ├── server.py
│   ├── tools.py
│   └── requirements.txt
│
├── rag-server/                        # new in Lab 8 (runs locally, not containerized)
│   ├── rag_server.py
│   ├── rag_http_server.py
│   ├── rag_pipeline.py
│   ├── rag_eval.py
│   ├── requirements.txt
│   ├── mcp-config.json
│   ├── tool-contracts.md
│   ├── retrieval-metrics.md
│   ├── rag-audit.jsonl
│   ├── corpus/
│   │   └── corpus.jsonl
│   └── chroma/
│
├── enrolment.db                       # shared database for all services
│
├── prompts/
│   ├── lab7/
│   └── lab8/                          # new in Lab 8
│       ├── implementation/
│       │   └── rag_implementation_prompt.txt
│       └── review/
│           ├── rag_review_prompt.txt
│           └── rag_reasoning_prompt.txt
│
└── reports/
    ├── tool-review.md                 # Lab 7
    ├── boundary-analysis.md           # Lab 7
    ├── integration-report.md          # Lab 7
    ├── run-report.md                  # Lab 7
    ├── rag-report.md                  # new in Lab 8
    └── rag-validation-report.md       # new in Lab 8
```

</details>

<details>
<summary>Create Project Workspace</summary>

```bash
# 1) Go to app root
cd enrolment-app-open-ai

# 2) RAG server (runs locally, not containerized)
mkdir -p rag-server/corpus
mkdir -p rag-server/chroma
touch rag-server/rag_server.py
touch rag-server/rag_http_server.py
touch rag-server/rag_pipeline.py
touch rag-server/rag_eval.py
touch rag-server/requirements.txt
touch rag-server/mcp-config.json
touch rag-server/tool-contracts.md
touch rag-server/retrieval-metrics.md
touch rag-server/rag-audit.jsonl
touch rag-server/corpus/corpus.jsonl

# 3) Enrolment service RAG integration
touch enrolment-service/routes/rag_mode.py
touch enrolment-service/services/rag_api.py

# 4) Frontend RAG tab
touch frontend-service/templates/tabs/rag.html

# 5) Agentic loop RAG mode
touch agentic_loop/collectors/rag_collector.py
touch agentic_loop/pipelines/rag_pipeline.py

# 6) Lab 8 prompts
mkdir -p prompts/lab8/implementation
mkdir -p prompts/lab8/review
touch prompts/lab8/implementation/rag_implementation_prompt.txt
touch prompts/lab8/review/rag_review_prompt.txt
touch prompts/lab8/review/rag_reasoning_prompt.txt

# 7) Lab 8 evidence reports
touch reports/rag-report.md
touch reports/rag-validation-report.md
```

</details>

---

## 4. RAG Pipeline Development

<details>
<summary>rag-server/requirements.txt</summary>

```text
mcp
chromadb
requests
```

**Install:**

```bash
cd enrolment-app-open-ai/rag-server
python3 -m pip install -r requirements.txt
```

</details>


<details>
<summary>Prompt assets</summary>

**`enrolment-app-open-ai/prompts/lab8/implementation/rag_implementation_prompt.txt`**
```text
Implement local RAG pipeline tools.

Constraints:
- local open-source stack only
- Python + chromadb + MCP
- no hosted commercial APIs

Required tools:
- refresh_corpus
- retrieve_context
- answer_question

Requirements:
- structured outputs
- structured errors
- citations
- confidence category
- audit logging

Maximum 60 words.
```

**`enrolment-app-open-ai/prompts/lab8/review/rag_review_prompt.txt`**
```text
Review RAG output quality.

Check:
- retrieval quality
- citation quality
- confidence justification
- unsupported claims

Return exactly:
Risk:
Correction:
Retest:

Maximum 35 words.
```

**`enrolment-app-open-ai/prompts/lab8/review/rag_reasoning_prompt.txt`**
```text
Evaluate RAG architecture.

Check:
- corpus design
- chunking
- embedding strategy
- vector store usage
- governance risks

Return exactly:
Strengths:
Risks:
Recommendations:

Maximum 45 words.
```

</details>

<details>
<summary>rag-server/rag_pipeline.py</summary>

```python
import json
import os
import sqlite3
import time
import uuid
import hashlib
from datetime import datetime, timezone
from pathlib import Path
from typing import Any

import chromadb
import requests

BASE_DIR = Path(__file__).resolve().parent
APP_DIR = BASE_DIR.parent
REPORTS_DIR = APP_DIR / "reports"
CORPUS_PATH = BASE_DIR / "corpus" / "corpus.jsonl"
AUDIT_PATH = BASE_DIR / "rag-audit.jsonl"
CHROMA_PATH = BASE_DIR / "chroma"
DATABASE_SERVICE_URL = os.getenv("DATABASE_SERVICE_URL", "http://database-service:5002")

DB_PATH_CANDIDATES = [
    APP_DIR / "database-service" / "data" / "enrolment.db",
    APP_DIR / "database-service" / "enrolment.db",
    APP_DIR / "enrolment.db",
]

REPORT_FILES = [
    "report.json",
    "run-report.md",
    "integration-report.md",
    "tool-review.md",
    "boundary-analysis.md",
]

COLLECTION_NAME = "student_enrolment_enterprise_context"
EMBED_VECTOR_SIZE = 256

_collection = None
_last_corpus_chunks: list[dict[str, Any]] = []


def now_iso() -> str:
    return datetime.now(timezone.utc).isoformat()


def resolve_db_path() -> Path:
    for path in DB_PATH_CANDIDATES:
        if path.exists():
            return path
    return DB_PATH_CANDIDATES[0]


def embed_texts(texts: list[str]) -> list[list[float]]:
    vectors: list[list[float]] = []

    for text in texts:
        values = [0.0] * EMBED_VECTOR_SIZE
        tokens = (text or "").lower().split()

        if not tokens:
            vectors.append(values)
            continue

        for token in tokens:
            digest = hashlib.sha256(token.encode("utf-8")).digest()
            for i, byte in enumerate(digest):
                idx = i % EMBED_VECTOR_SIZE
                values[idx] += (byte / 255.0) - 0.5

        norm = sum(v * v for v in values) ** 0.5
        if norm > 0:
            values = [v / norm for v in values]

        vectors.append(values)

    return vectors


def get_collection():
    global _collection
    if _collection is None:
        client = chromadb.PersistentClient(path=str(CHROMA_PATH))
        _collection = client.get_or_create_collection(name=COLLECTION_NAME)
    return _collection


def reset_collection() -> None:
    global _collection
    client = chromadb.PersistentClient(path=str(CHROMA_PATH))
    try:
        client.delete_collection(name=COLLECTION_NAME)
    except Exception:
        pass
    _collection = client.get_or_create_collection(name=COLLECTION_NAME)


def append_audit(
    tool_name: str,
    tool_input: dict[str, Any],
    tool_output: dict[str, Any],
    validation_status: str,
    outcome: str,
    start_time: float,
) -> None:
    AUDIT_PATH.parent.mkdir(parents=True, exist_ok=True)
    duration_ms = int((time.time() - start_time) * 1000)
    record = {
        "request_id": str(uuid.uuid4()),
        "trace_id": str(uuid.uuid4()),
        "tool_name": tool_name,
        "tool_input": tool_input,
        "tool_output": tool_output,
        "timestamp": now_iso(),
        "duration_ms": duration_ms,
        "validation_status": validation_status,
        "outcome": outcome,
    }
    with AUDIT_PATH.open("a", encoding="utf-8") as f:
        f.write(json.dumps(record) + "\n")


def chunk_text(text: str, max_words: int = 80) -> list[str]:
    words = text.split()
    if not words:
        return []
    chunks = []
    for i in range(0, len(words), max_words):
        chunk = " ".join(words[i : i + max_words]).strip()
        if chunk:
            chunks.append(chunk)
    return chunks


def load_database_chunks() -> list[dict[str, Any]]:
    db_path = resolve_db_path()
    if not db_path.exists():
        # In containers, rag-server may not have direct filesystem access to SQLite.
        # Fallback to database-service HTTP API so tier_1 facts remain available.
        try:
            response = requests.get(f"{DATABASE_SERVICE_URL}/students", timeout=10)
            response.raise_for_status()
            students = response.json()

            chunks: list[dict[str, Any]] = [
                {
                    "chunk_id": "db_service_student_count",
                    "source_id": "database-service:/students",
                    "authority_tier": "tier_1",
                    "text": f"Student count is {len(students)}.",
                    "metadata": {"source_type": "database_service", "metric": "count"},
                    "indexed_at": now_iso(),
                }
            ]

            for row in students[:500]:
                sid = row.get("student_id", "unknown")
                sname = row.get("student_name", "unknown")
                subject = row.get("subject_code", "unknown")
                chunks.append(
                    {
                        "chunk_id": f"db_service_student_{sid}",
                        "source_id": "database-service:/students",
                        "authority_tier": "tier_1",
                        "text": (
                            f"Student record: student_id={sid}, "
                            f"student_name={sname}, subject_code={subject}."
                        ),
                        "metadata": {"source_type": "database_service", "table": "students"},
                        "indexed_at": now_iso(),
                    }
                )

            return chunks
        except Exception as exc:
            return [
                {
                    "chunk_id": "db_missing",
                    "source_id": str(db_path.relative_to(APP_DIR)) if db_path.is_absolute() else str(db_path),
                    "authority_tier": "tier_1",
                    "text": (
                        f"Database file not found: {db_path}. "
                        f"database-service fallback failed: {exc}"
                    ),
                    "metadata": {"source_type": "database", "exists": False},
                    "indexed_at": now_iso(),
                }
            ]

    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    chunks: list[dict[str, Any]] = []
    try:
        tables = [
            row["name"]
            for row in conn.execute("SELECT name FROM sqlite_master WHERE type='table'").fetchall()
        ]
        chunks.append(
            {
                "chunk_id": "db_schema",
                "source_id": str(db_path.relative_to(APP_DIR)),
                "authority_tier": "tier_1",
                "text": f"Database tables: {', '.join(tables)}",
                "metadata": {"source_type": "database", "tables": tables},
                "indexed_at": now_iso(),
            }
        )

        if "students" in tables:
            row = conn.execute("SELECT COUNT(*) AS count FROM students").fetchone()
            count = row["count"] if row else 0
            chunks.append(
                {
                    "chunk_id": "db_student_count",
                    "source_id": str(db_path.relative_to(APP_DIR)),
                    "authority_tier": "tier_1",
                    "text": f"Student count is {count}.",
                    "metadata": {"source_type": "database", "table": "students", "metric": "count"},
                    "indexed_at": now_iso(),
                }
            )

            for row in conn.execute(
                "SELECT student_id, student_name, subject_code FROM students ORDER BY student_id LIMIT 500"
            ).fetchall():
                r = dict(row)
                chunks.append(
                    {
                        "chunk_id": f"db_student_{r['student_id']}",
                        "source_id": str(db_path.relative_to(APP_DIR)),
                        "authority_tier": "tier_1",
                        "text": (
                            f"Student record: student_id={r['student_id']}, "
                            f"student_name={r['student_name']}, subject_code={r['subject_code']}."
                        ),
                        "metadata": {"source_type": "database", "table": "students"},
                        "indexed_at": now_iso(),
                    }
                )
    finally:
        conn.close()

    return chunks


def load_report_chunks() -> list[dict[str, Any]]:
    chunks: list[dict[str, Any]] = []
    for name in REPORT_FILES:
        path = REPORTS_DIR / name
        if not path.exists():
            continue

        text = ""
        try:
            if path.suffix == ".json":
                text = json.dumps(json.loads(path.read_text(encoding="utf-8")), indent=2)
            else:
                text = path.read_text(encoding="utf-8", errors="ignore")
        except Exception:
            text = path.read_text(encoding="utf-8", errors="ignore")

        for i, chunk in enumerate(chunk_text(text), start=1):
            chunks.append(
                {
                    "chunk_id": f"{path.stem}_{i}",
                    "source_id": f"reports/{name}",
                    "authority_tier": "tier_2",
                    "text": chunk,
                    "metadata": {"source_type": "report", "file": name},
                    "indexed_at": now_iso(),
                }
            )

    return chunks


def load_repository_chunks() -> list[dict[str, Any]]:
    ignored = {".git", ".venv", "__pycache__", "node_modules", "chroma"}
    files: list[str] = []

    for root, dirs, filenames in os.walk(APP_DIR, topdown=True, followlinks=False, onerror=lambda e: None):
        # Prune ignored and symlinked directories to avoid scanning protected mounts.
        pruned_dirs: list[str] = []
        for directory_name in dirs:
            if directory_name in ignored:
                continue
            directory_path = Path(root) / directory_name
            try:
                if directory_path.is_symlink():
                    continue
            except OSError:
                continue
            pruned_dirs.append(directory_name)
        dirs[:] = pruned_dirs

        for filename in filenames:
            file_path = Path(root) / filename
            try:
                if file_path.is_symlink():
                    continue
                rel = file_path.relative_to(APP_DIR)
                files.append(str(rel).replace("\\", "/"))
            except (OSError, ValueError):
                continue

    text = "Repository files include: " + ", ".join(sorted(files[:400]))
    return [
        {
            "chunk_id": "repo_index",
            "source_id": "repository",
            "authority_tier": "tier_3",
            "text": text,
            "metadata": {"source_type": "repository", "file_count": len(files)},
            "indexed_at": now_iso(),
        }
    ]


def build_corpus() -> list[dict[str, Any]]:
    chunks: list[dict[str, Any]] = []
    chunks.extend(load_database_chunks())
    chunks.extend(load_report_chunks())
    chunks.extend(load_repository_chunks())
    return chunks


def write_corpus(chunks: list[dict[str, Any]]) -> None:
    CORPUS_PATH.parent.mkdir(parents=True, exist_ok=True)
    with CORPUS_PATH.open("w", encoding="utf-8") as f:
        for chunk in chunks:
            f.write(json.dumps(chunk) + "\n")


def read_corpus() -> list[dict[str, Any]]:
    if not CORPUS_PATH.exists():
        return []

    chunks: list[dict[str, Any]] = []
    with CORPUS_PATH.open("r", encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                chunks.append(json.loads(line))
            except json.JSONDecodeError:
                continue
    return chunks


def lexical_fallback_retrieve(query: str, k: int) -> list[dict[str, Any]]:
    corpus = _last_corpus_chunks or read_corpus()
    query_tokens = set((query or "").lower().split())
    tier_weight = {"tier_1": 3, "tier_2": 2, "tier_3": 1}

    scored = []
    for chunk in corpus:
        text = chunk.get("text", "")
        text_tokens = set(text.lower().split())
        overlap = len(query_tokens.intersection(text_tokens))
        scored.append(
            {
                "rank": 0,
                "chunk_id": chunk.get("chunk_id"),
                "source_id": chunk.get("source_id"),
                "authority_tier": chunk.get("authority_tier"),
                "distance": None,
                "text": text,
                "_score": overlap,
            }
        )

    scored.sort(
        key=lambda r: (
            tier_weight.get(r.get("authority_tier"), 0),
            r.get("_score", 0),
        ),
        reverse=True,
    )

    top = scored[: max(k, 1)]
    for i, row in enumerate(top, start=1):
        row["rank"] = i
        row.pop("_score", None)
    return top


def refresh_corpus(caller: str = "student") -> dict[str, Any]:
    global _last_corpus_chunks
    start = time.time()
    try:
        chunks = build_corpus()
        _last_corpus_chunks = chunks
        write_corpus(chunks)
        vector_store_status = "ready"
        vector_store_error = None

        try:
            reset_collection()
            collection = get_collection()

            if chunks:
                ids = [c["chunk_id"] for c in chunks]
                docs = [c["text"] for c in chunks]
                metas = [
                    {
                        "source_id": c["source_id"],
                        "authority_tier": c["authority_tier"],
                        "indexed_at": c["indexed_at"],
                    }
                    for c in chunks
                ]
                embeddings = embed_texts(docs)
                collection.add(ids=ids, documents=docs, metadatas=metas, embeddings=embeddings)
        except Exception as exc:
            vector_store_status = "degraded"
            vector_store_error = str(exc)

        output = {
            "status": "success",
            "caller": caller,
            "chunk_count": len(chunks),
            "collection": COLLECTION_NAME,
            "corpus_path": str(CORPUS_PATH),
            "vector_store_status": vector_store_status,
        }
        if vector_store_error:
            output["vector_store_error"] = vector_store_error
        append_audit("refresh_corpus", {"caller": caller}, output, "pass", "corpus_refreshed", start)
        return output
    except Exception as exc:
        output = {"status": "error", "error": str(exc)}
        append_audit("refresh_corpus", {"caller": caller}, output, "fail", "error", start)
        return output


def retrieve_context(query: str, k: int = 5, caller: str = "student") -> dict[str, Any]:
    start = time.time()
    try:
        retrieval_mode = "vector"
        ranked = []

        try:
            collection = get_collection()
            if collection.count() == 0:
                refreshed = refresh_corpus(caller="auto_refresh")
                if refreshed.get("status") != "success":
                    raise RuntimeError("empty_collection")

            query_embedding = embed_texts([query])
            results = collection.query(query_embeddings=query_embedding, n_results=k)

            ids = (results.get("ids") or [[]])[0]
            docs = (results.get("documents") or [[]])[0]
            metas = (results.get("metadatas") or [[]])[0]
            distances = (results.get("distances") or [[]])[0]

            for i, chunk_id in enumerate(ids):
                row_meta = metas[i] if i < len(metas) and isinstance(metas[i], dict) else {}
                ranked.append(
                    {
                        "rank": i + 1,
                        "chunk_id": chunk_id,
                        "source_id": row_meta.get("source_id"),
                        "authority_tier": row_meta.get("authority_tier"),
                        "distance": distances[i] if i < len(distances) else None,
                        "text": docs[i] if i < len(docs) else "",
                    }
                )

            tier_weight = {"tier_1": 3, "tier_2": 2, "tier_3": 1}
            ranked.sort(
                key=lambda x: (
                    tier_weight.get(x.get("authority_tier"), 0),
                    -(x.get("distance") if isinstance(x.get("distance"), (int, float)) else 1e9),
                ),
                reverse=True,
            )
        except Exception:
            retrieval_mode = "lexical_fallback"
            if not _last_corpus_chunks and not CORPUS_PATH.exists():
                refreshed = refresh_corpus(caller="auto_refresh")
                if refreshed.get("status") != "success":
                    return {"status": "error", "error": "corpus_unavailable"}
            ranked = lexical_fallback_retrieve(query, k)

        output = {
            "status": "success",
            "query": query,
            "caller": caller,
            "k": k,
            "retrieval_mode": retrieval_mode,
            "results": ranked,
        }

        append_audit(
            "retrieve_context",
            {"query": query, "k": k, "caller": caller},
            {"result_count": len(ranked), "chunk_ids": [r["chunk_id"] for r in ranked]},
            "pass",
            "context_retrieved",
            start,
        )
        return output
    except Exception as exc:
        output = {"status": "error", "error": str(exc), "query": query}
        append_audit(
            "retrieve_context",
            {"query": query, "k": k, "caller": caller},
            output,
            "fail",
            "error",
            start,
        )
        return output


def confidence_from_results(results: list[dict[str, Any]]) -> str:
    if not results:
        return "Unknown"
    tier_1 = sum(1 for r in results if r.get("authority_tier") == "tier_1")
    tier_2 = sum(1 for r in results if r.get("authority_tier") == "tier_2")
    if tier_1 >= 2 and len(results) >= 3:
        return "High"
    if tier_1 >= 1 or tier_2 >= 2:
        return "Medium"
    return "Low"


def extract_student_records(results: list[dict[str, Any]]) -> list[dict[str, str]]:
    records: list[dict[str, str]] = []
    for row in results:
        text = row.get("text", "")
        if "Student record:" not in text:
            continue

        payload = text.split("Student record:", 1)[1].strip().rstrip(".")
        parts = [p.strip() for p in payload.split(",")]
        values: dict[str, str] = {}
        for part in parts:
            if "=" not in part:
                continue
            key, value = part.split("=", 1)
            values[key.strip()] = value.strip()

        if values.get("student_id") and values.get("subject_code"):
            records.append(values)

    # Keep stable ordering by numeric student_id when possible.
    def key_fn(item: dict[str, str]):
        sid = item.get("student_id", "")
        return (0, int(sid)) if sid.isdigit() else (1, sid)

    records.sort(key=key_fn)
    return records


def deterministic_answer(query: str, results: list[dict[str, Any]]) -> str | None:
    q = (query or "").lower()
    records = extract_student_records(results)
    if not records:
        return None

    if "student ids and subject codes" in q or "student id and subject code" in q:
        lines = [f"{r.get('student_id')} -> {r.get('subject_code')}" for r in records]
        return "Answer:\n" + "\n".join(lines)

    if "how many students" in q or "student count" in q:
        return f"Answer:\nThere are {len(records)} students in the retrieved evidence."

    return None


def generate_with_ollama(query: str, context: str) -> str:
    model_name = os.getenv("OLLAMA_MODEL", "qwen2.5:0.5b")
    ollama_generate_url = os.getenv("OLLAMA_GENERATE_URL", "http://host.docker.internal:11434/api/generate")
    prompt = f"""
You are a retrieval-grounded assistant.
Use only the provided context.
If evidence is missing, return exactly: Insufficient evidence.

QUESTION:
{query}

CONTEXT:
{context}

Return exactly:
Answer:
<answer>

Evidence:
<summary>
"""

    try:
        resp = requests.post(
            ollama_generate_url,
            json={"model": model_name, "prompt": prompt, "stream": False},
            timeout=120,
        )
        resp.raise_for_status()
        return resp.json().get("response", "Insufficient evidence.")
    except Exception as exc:
        return f"Ollama unavailable: {exc}"


def answer_question(query: str, k: int = 5, caller: str = "student") -> dict[str, Any]:
    start = time.time()
    retrieval = retrieve_context(query=query, k=k, caller=caller)
    if retrieval.get("status") != "success":
        output = {"status": "error", "query": query, "error": retrieval.get("error", "retrieval_failed")}
        append_audit(
            "answer_question",
            {"query": query, "k": k, "caller": caller},
            output,
            "fail",
            "retrieval_failed",
            start,
        )
        return output

    results = retrieval.get("results", [])
    context = "\n\n".join(r.get("text", "") for r in results)
    answer = deterministic_answer(query, results)
    if answer is None:
        answer = generate_with_ollama(query, context)

    citations = [
        {
            "chunk_id": r.get("chunk_id"),
            "source_id": r.get("source_id"),
            "authority_tier": r.get("authority_tier"),
        }
        for r in results
    ]

    confidence = confidence_from_results(results)
    output = {
        "status": "success",
        "query": query,
        "answer": answer,
        "citations": citations,
        "confidence_category": confidence,
        "retrieval_summary": {
            "k": k,
            "retrieved_count": len(results),
            "top_chunk": results[0].get("chunk_id") if results else None,
        },
    }

    append_audit(
        "answer_question",
        {"query": query, "k": k, "caller": caller},
        {"confidence_category": confidence, "citation_count": len(citations)},
        "pass",
        "answer_generated",
        start,
    )
    return output


if __name__ == "__main__":
    print(json.dumps(refresh_corpus(), indent=2))
    print(json.dumps(retrieve_context("students enrolled in ASD101", 5), indent=2))
    print(json.dumps(answer_question("Which students are enrolled in ASD101?", 5), indent=2))
```

**Run:**

```bash
cd enrolment-app-open-ai/rag-server
python rag_pipeline.py
```

**Expected:**
- `refresh_corpus` returns `status: success`
- `retrieve_context` returns `results`
- `answer_question` returns answer + citations + confidence

</details>

<details>
<summary>rag-server/rag_server.py</summary>

```python
from mcp.server.fastmcp import FastMCP

from rag_pipeline import answer_question as answer_question_impl
from rag_pipeline import refresh_corpus as refresh_corpus_impl
from rag_pipeline import retrieve_context as retrieve_context_impl

mcp = FastMCP("Student Enrolment RAG MCP")
AVAILABLE_TOOLS = ["refresh_corpus", "retrieve_context", "answer_question"]


@mcp.tool()
def refresh_corpus(caller: str = "student"):
    return refresh_corpus_impl(caller=caller)


@mcp.tool()
def retrieve_context(query: str, k: int = 5, caller: str = "student"):
    return retrieve_context_impl(query=query, k=k, caller=caller)


@mcp.tool()
def answer_question(query: str, k: int = 5, caller: str = "student"):
    return answer_question_impl(query=query, k=k, caller=caller)


if __name__ == "__main__":
    print("Starting Student Enrolment RAG MCP Server...")
    print("Server status: RUNNING")
    print("Interact with RAG tools from a second terminal.")
    print("Available tools:")
    for tool in AVAILABLE_TOOLS:
        print(f"- {tool}")
    mcp.run()
```

**Run:**

```bash
cd enrolment-app-open-ai/rag-server
python rag_server.py
```

**Expected output:**
```text
Starting Student Enrolment RAG MCP Server...
Server status: RUNNING
Interact with RAG tools from a second terminal.
Available tools:
- refresh_corpus
- retrieve_context
- answer_question
```

</details>

<details>
<summary>rag-server/rag_http_server.py</summary>

```python
import json
import os
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer

from rag_pipeline import answer_question, refresh_corpus, retrieve_context


class RAGHandler(BaseHTTPRequestHandler):
    def _send_json(self, status_code: int, payload: dict):
        response = json.dumps(payload).encode("utf-8")
        self.send_response(status_code)
        self.send_header("Content-Type", "application/json")
        self.send_header("Content-Length", str(len(response)))
        self.end_headers()
        self.wfile.write(response)

    def _read_json(self):
        content_length = int(self.headers.get("Content-Length", "0"))
        if content_length == 0:
            return {}
        raw = self.rfile.read(content_length)
        if not raw:
            return {}
        return json.loads(raw.decode("utf-8"))

    def do_GET(self):
        if self.path == "/health":
            self._send_json(200, {"status": "ok", "service": "rag-server"})
            return
        self._send_json(404, {"status": "error", "error": "not_found"})

    def do_POST(self):
        try:
            payload = self._read_json()
        except Exception as exc:
            self._send_json(400, {"status": "error", "error": f"invalid_json: {exc}"})
            return

        try:
            if self.path == "/refresh":
                caller = (payload.get("caller") or "student").strip() or "student"
                result = refresh_corpus(caller=caller)
                self._send_json(200 if result.get("status") == "success" else 500, result)
                return

            if self.path == "/retrieve":
                query = (payload.get("query") or "").strip()
                if not query:
                    self._send_json(400, {"status": "error", "error": "query is required"})
                    return
                k = int(payload.get("k", 5))
                caller = (payload.get("caller") or "student").strip() or "student"
                result = retrieve_context(query=query, k=k, caller=caller)
                self._send_json(200 if result.get("status") == "success" else 500, result)
                return

            if self.path == "/answer":
                query = (payload.get("query") or "").strip()
                if not query:
                    self._send_json(400, {"status": "error", "error": "query is required"})
                    return
                k = int(payload.get("k", 5))
                caller = (payload.get("caller") or "student").strip() or "student"
                result = answer_question(query=query, k=k, caller=caller)
                self._send_json(200 if result.get("status") == "success" else 500, result)
                return

            self._send_json(404, {"status": "error", "error": "not_found"})
        except Exception as exc:
            self._send_json(500, {"status": "error", "error": str(exc)})


def main():
    host = "0.0.0.0"
    port = int(os.getenv("PORT", "5003"))
    server = ThreadingHTTPServer((host, port), RAGHandler)
    print(f"RAG HTTP server running on {host}:{port}")
    server.serve_forever()


if __name__ == "__main__":
    main()
```

</details>

<details>
<summary>rag-server/rag_eval.py</summary>

```python
from pathlib import Path

from rag_pipeline import retrieve_context

METRICS_PATH = Path(__file__).resolve().parent / "retrieval-metrics.md"

BENCHMARKS = [
    {
        "query": "students enrolled in ASD101",
        "relevant_keywords": ["ASD101"],
        "expected_relevant": 2,
    },
    {
        "query": "student count",
        "relevant_keywords": ["student", "count"],
        "expected_relevant": 1,
    },
    {
        "query": "CI report status",
        "relevant_keywords": ["report", "workflow", "run"],
        "expected_relevant": 1,
    },
]


def is_relevant(text: str, keywords: list[str]) -> bool:
    lowered = text.lower()
    return any(keyword.lower() in lowered for keyword in keywords)


def evaluate_query(benchmark: dict) -> dict:
    response = retrieve_context(benchmark["query"], 5)
    results = response.get("results", [])[:5]

    relevant = [r for r in results if is_relevant(r.get("text", ""), benchmark["relevant_keywords"])]

    retrieved_count = len(results)
    precision_at_5 = len(relevant) / 5
    raw_recall_at_5 = len(relevant) / max(benchmark["expected_relevant"], 1)
    recall_at_5 = min(1.0, raw_recall_at_5)

    return {
        "query": benchmark["query"],
        "retrieved_chunk_ids": [r.get("chunk_id") for r in results],
        "relevant_chunk_ids": [r.get("chunk_id") for r in relevant],
        "p_at_5": precision_at_5,
        "r_at_5": recall_at_5,
    }


def write_metrics_report(results: list[dict]) -> None:
    lines = ["# Retrieval Metrics", ""]
    for result in results:
        lines.append(f"## {result['query']}")
        lines.append(f"- Retrieved: {result['retrieved_chunk_ids']}")
        lines.append(f"- Relevant: {result['relevant_chunk_ids']}")
        lines.append(f"- P@5: {result['p_at_5']}")
        lines.append(f"- R@5: {result['r_at_5']}")
        lines.append("")
    METRICS_PATH.write_text("\n".join(lines), encoding="utf-8")


def main() -> None:
    results = []
    for benchmark in BENCHMARKS:
        result = evaluate_query(benchmark)
        results.append(result)
        print("Query:", result["query"])
        print("Retrieved:", result["retrieved_chunk_ids"])
        print("Relevant:", result["relevant_chunk_ids"])
        print("P@5:", result["p_at_5"])
        print("R@5:", result["r_at_5"])
        print("---")
    write_metrics_report(results)


if __name__ == "__main__":
    main()
```

**Database Setup:**

The RAG server uses the shared `enrolment.db` located in the app root (`enrolment-app-open-ai/enrolment.db`), which was created in Labs 03-04 and populated with 10 student records. The `resolve_db_path()` function in `rag_pipeline.py` searches for the database in multiple candidate locations and uses the first one found.

**Run:**

```bash
cd enrolment-app-open-ai/rag-server

# Execute local evaluation (uses shared database in app root)
python rag_eval.py
```

**Expected:** prints `P@5` and `R@5` for the 3 benchmark queries and writes the same results to `retrieval-metrics.md`.

</details>

<details>
<summary>rag-server/mcp-config.json</summary>

```json
{
  "mcpServers": {
    "student-enrolment-rag": {
      "command": "python",
      "args": ["rag_server.py"]
    }
  }
}
```

Used by MCP clients (e.g. Claude Desktop, VS Code MCP extension) to launch `rag_server.py` as a stdio MCP server exposing `refresh_corpus`, `retrieve_context`, and `answer_question`.

</details>

<details>
<summary>rag-server/tool-contracts.md</summary>

```markdown
# RAG Tool Contracts

## refresh_corpus
- Purpose: rebuild corpus and vector index
- Input: `caller` (optional)
- Output: `status`, `chunk_count`, `collection`, `corpus_path` or `error`
- Policy class: read + index update

## retrieve_context
- Purpose: retrieve relevant chunks
- Input: `query` (required), `k` (optional), `caller` (optional)
- Output: `status`, `results[]` with `chunk_id`, `source_id`, `authority_tier`, `distance`, `text`
- Policy class: read

## answer_question
- Purpose: answer from retrieved context only
- Input: `query` (required), `k` (optional), `caller` (optional)
- Output: `answer`, `citations[]`, `confidence_category`, `retrieval_summary` or `error`
- Policy class: read + grounded response
```

</details>

<details>
<summary>UI Integration (RAG Mode)</summary>

- `frontend-service/css/styles.css` **provided**
- `frontend-service/templates/index.html` **provided**

**`frontend-service/templates/tabs/rag.html`**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>RAG Tab</title>
    <link rel="stylesheet" href="../css/styles.css">
</head>
<body>
<section class="card">
    <h2>RAG Mode</h2>
    <p>Refresh corpus, retrieve context, and generate grounded answers with citations.</p>

    <div class="feature-toggle-row">
        <label class="toggle-switch" for="rag-mode-toggle">
            <input id="rag-mode-toggle" type="checkbox" checked>
            <span class="toggle-label">RAG Enabled</span>
        </label>
        <span id="rag-mode-state" class="feature-state feature-on">ON</span>
    </div>

    <form id="rag-refresh-form" class="stack-form">
        <button type="submit" class="rag-action">Refresh Corpus</button>
    </form>

    <form id="rag-retrieve-form" class="stack-form">
        <label for="rag-retrieve-query">Query</label>
        <input
            id="rag-retrieve-query"
            name="query"
            required
            placeholder="Example: students enrolled in ASD101"
        >
        <p class="rag-helper">Hint: use short retrieval phrases.</p>
        <button type="submit" class="rag-action">Retrieve Context</button>
    </form>

    <form id="rag-answer-form" class="stack-form">
        <label for="rag-answer-query">Question</label>
        <input
            id="rag-answer-query"
            name="query"
            required
            placeholder="Example: Which students are enrolled in ASD101?"
        >
        <p class="rag-helper">Hint: ask a full question for grounded answers.</p>
        <button type="submit" class="rag-action">Answer with Citations</button>
    </form>

    <div id="rag-results" class="panel panel-mcp">RAG responses will appear here.</div>
</section>

<script>
const apiBase = "http://localhost:5001";
const output = document.getElementById("rag-results");
const ragModeToggle = document.getElementById("rag-mode-toggle");
const ragModeState = document.getElementById("rag-mode-state");
const RAG_STORAGE_KEY = "rag_mode_enabled";

function isRagEnabled() {
    return ragModeToggle.checked;
}

function renderRagState() {
    if (isRagEnabled()) {
        ragModeState.textContent = "ON";
        ragModeState.classList.add("feature-on");
        ragModeState.classList.remove("feature-off");
    } else {
        ragModeState.textContent = "OFF";
        ragModeState.classList.add("feature-off");
        ragModeState.classList.remove("feature-on");
    }
}

function renderRagDisabledMessage() {
    output.innerHTML = "<p>RAG Mode is OFF. Enable RAG Mode to run RAG tools.</p>";
}

function saveRagMode() {
    localStorage.setItem(RAG_STORAGE_KEY, String(isRagEnabled()));
}

function loadRagMode() {
    const persisted = localStorage.getItem(RAG_STORAGE_KEY);
    if (persisted === null) {
        ragModeToggle.checked = true;
    } else {
        ragModeToggle.checked = persisted === "true";
    }
    renderRagState();
}

function renderJson(title, payload) {
    output.innerHTML = `<h3>${title}</h3><pre>${JSON.stringify(payload, null, 2)}</pre>`;
}

async function postForm(path, data) {
    if (!isRagEnabled()) {
        renderRagDisabledMessage();
        return { status: "error", error: "RAG Mode is OFF" };
    }

    const res = await fetch(`${apiBase}${path}`, {
        method: "POST",
        headers: {
            "Content-Type": "application/x-www-form-urlencoded",
            "X-RAG-Mode": isRagEnabled() ? "on" : "off",
        },
        body: new URLSearchParams(data),
    });
    const text = await res.text();
    try {
        return JSON.parse(text);
    } catch {
        return { raw: text };
    }
}

ragModeToggle.addEventListener("change", () => {
    saveRagMode();
    renderRagState();

    if (!isRagEnabled()) {
        renderRagDisabledMessage();
    }
});

document.getElementById("rag-refresh-form").addEventListener("submit", async (e) => {
    e.preventDefault();
    const payload = await postForm("/rag/refresh", {});
    renderJson("RAG Tool: refresh_corpus", payload);
});

document.getElementById("rag-retrieve-form").addEventListener("submit", async (e) => {
    e.preventDefault();
    const query = document.getElementById("rag-retrieve-query").value.trim();
    const payload = await postForm("/rag/retrieve", { query, k: "5" });
    renderJson("RAG Tool: retrieve_context", payload);
});

document.getElementById("rag-answer-form").addEventListener("submit", async (e) => {
    e.preventDefault();
    const query = document.getElementById("rag-answer-query").value.trim();
    const payload = await postForm("/rag/answer", { query, k: "5" });
    renderJson("RAG Tool: answer_question", payload);
});

loadRagMode();
if (!isRagEnabled()) {
    renderRagDisabledMessage();
}
</script>

</body>
</html>
```


</details>

<details>
<summary>Backend Integration (RAG Integration Endpoints)</summary>

Update these backend files:

```text
enrolment-app-open-ai/enrolment-service/app.py
enrolment-app-open-ai/enrolment-service/routes/rag_mode.py
enrolment-app-open-ai/enrolment-service/services/rag_api.py
enrolment-app-open-ai/docker-compose.yml
```

<details>
<summary>enrolment-service/app.py</summary>

```python
from pathlib import Path
import sys

from flask import Flask
from flask_cors import CORS


BASE_DIR = Path(__file__).resolve().parent
if str(BASE_DIR) not in sys.path:
    sys.path.insert(0, str(BASE_DIR))

from routes.ai_mode import ai_mode_bp
from routes.mcp_mode import mcp_bp
from routes.normal_ui import normal_ui_bp
from routes.rag_mode import rag_bp


def create_app():
    app = Flask(__name__)
    CORS(app)

    app.register_blueprint(normal_ui_bp)
    app.register_blueprint(ai_mode_bp)
    app.register_blueprint(mcp_bp)
    app.register_blueprint(rag_bp)

    return app


app = create_app()


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5001, debug=True)
```

</details>

<details>
<summary>enrolment-service/routes/rag_mode.py</summary>

```python
from flask import Blueprint, request
import requests

from services.rag_api import call_rag_service, rag_disabled_response, rag_mode_is_enabled


rag_bp = Blueprint("rag_mode", __name__)


@rag_bp.post("/rag/refresh")
def rag_refresh():
    if not rag_mode_is_enabled(request):
        return rag_disabled_response()

    caller = request.form.get("caller", "student").strip() or "student"
    try:
        payload = call_rag_service("/refresh", {"caller": caller})
        return payload, 200
    except requests.RequestException as exc:
        return {"status": "error", "error": str(exc)}, 503


@rag_bp.post("/rag/retrieve")
def rag_retrieve():
    if not rag_mode_is_enabled(request):
        return rag_disabled_response()

    query = request.form.get("query", "").strip()
    k = int(request.form.get("k", "5"))
    if not query:
        return {"status": "error", "error": "query is required"}, 400

    try:
        payload = call_rag_service("/retrieve", {"query": query, "k": k, "caller": "student"})
        return payload, 200
    except requests.RequestException as exc:
        return {"status": "error", "error": str(exc)}, 503


@rag_bp.post("/rag/answer")
def rag_answer():
    if not rag_mode_is_enabled(request):
        return rag_disabled_response()

    query = request.form.get("query", "").strip()
    k = int(request.form.get("k", "5"))
    if not query:
        return {"status": "error", "error": "query is required"}, 400

    try:
        payload = call_rag_service("/answer", {"query": query, "k": k, "caller": "student"})
        return payload, 200
    except requests.RequestException as exc:
        return {"status": "error", "error": str(exc)}, 503
```

</details>

<details>
<summary>enrolment-service/services/rag_api.py</summary>

```python
import os

import requests


RAG_SERVICE_URL = os.getenv("RAG_SERVICE_URL", "http://rag-server:5003")
RAG_ENABLED = os.getenv("RAG_ENABLED", "true").strip().lower() in ("1", "true", "yes", "on")

try:
    RAG_SERVICE_TIMEOUT_SECONDS = int(os.getenv("RAG_SERVICE_TIMEOUT_SECONDS", "180"))
except ValueError:
    RAG_SERVICE_TIMEOUT_SECONDS = 180


def rag_mode_is_enabled(req) -> bool:
    if not RAG_ENABLED:
        return False

    mode_header = req.headers.get("X-RAG-Mode", "on").strip().lower()
    return mode_header in ("1", "true", "yes", "on")


def rag_disabled_response():
    return {"status": "error", "error": "RAG mode is disabled."}, 403


def call_rag_service(path: str, payload: dict):
    response = requests.post(
        f"{RAG_SERVICE_URL}{path}",
        json=payload,
        timeout=RAG_SERVICE_TIMEOUT_SECONDS,
    )

    try:
        data = response.json()
    except ValueError:
        response.raise_for_status()
        return {}

    if response.status_code >= 400:
        raise requests.HTTPError(
            f"RAG service {path} failed with status {response.status_code}: {data}",
            response=response,
        )

    return data
```

</details>

<details>
<summary>docker-compose.yml</summary>

```yaml
services:
  frontend-service:
    build:
      context: ./frontend-service
    container_name: frontend-service
    ports:
      - "8080:80"
    depends_on:
      - enrolment-service
    restart: unless-stopped

  enrolment-service:
    build:
      context: .
      dockerfile: enrolment-service/Dockerfile
    container_name: enrolment-service
    ports:
      - "5001:5001"
    environment:
      DATABASE_SERVICE_URL: http://database-service:5002
      OLLAMA_BASE_URL: http://host.docker.internal:11434/v1
      OLLAMA_MODEL: qwen2.5:0.5b
      MCP_ENABLED: "true"
      RAG_SERVICE_URL: http://host.docker.internal:5003
      RAG_ENABLED: "true"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      - database-service
    restart: unless-stopped

  database-service:
    build:
      context: ./database-service
    container_name: database-service
    ports:
      - "5002:5002"
    volumes:
      - database_data:/app/data
    restart: unless-stopped

volumes:
  database_data:
```

**Note:** RAG server is NOT containerized. It runs locally and is accessed by containerized services via `http://host.docker.internal:5003`, similar to how Ollama and MCP server are accessed.

</details>

</details>

---

## 5. RAG Validation

<details>
<summary>Terminal Testing</summary>

**Terminal A - Start RAG MCP server (for MCP testing):**
```bash
cd enrolment-app-open-ai/rag-server
python3 rag_server.py
```

**Expected output:**
```text
Starting Student Enrolment RAG MCP Server...
Server status: RUNNING
Available tools:
- refresh_corpus
- retrieve_context
- answer_question
```

Keep Terminal A running.

**Terminal B - Test RAG tools via Python:**
```bash
cd enrolment-app-open-ai/rag-server

# 1. Refresh corpus
python3 -c "from rag_pipeline import refresh_corpus; import json; print(json.dumps(refresh_corpus(), indent=2))"

# 2. Retrieve context
python3 -c "from rag_pipeline import retrieve_context; import json; print(json.dumps(retrieve_context('students enrolled in ASD101', 5), indent=2))"

# 3. Answer question
python3 -c "from rag_pipeline import answer_question; import json; print(json.dumps(answer_question('Which students are enrolled in ASD101?', 5), indent=2))"

# 4. Run evaluation
python3 rag_eval.py
```

**Pass criteria:**
- `refresh_corpus` returns `status: success` with `chunk_count > 0`
- `retrieve_context` returns `results[]` with `chunk_id`, `source_id`, `authority_tier`, `text`
- `answer_question` returns `answer`, `citations[]`, `confidence_category`
- `rag_eval.py` outputs P@5, R@5 metrics for 3 test queries

**Record in `reports/rag-report.md`:**
- Query used
- Retrieval results count
- Answer confidence category
- Citations provided
- P@5 and R@5 metrics

</details>

<details>
<summary>UI Testing</summary>

**IMPORTANT:** RAG server must be running locally BEFORE starting Docker services (similar to MCP server in Lab 07).

**Step 1 - Start RAG HTTP Server (Terminal 1):**
```bash
cd enrolment-app-open-ai/rag-server
python3 rag_http_server.py
```

**Expected output:**
```text
RAG HTTP server running on 0.0.0.0:5003
```

**Keep Terminal 1 running** - RAG server must stay active for Docker services to connect.

**Step 2 - Verify RAG server health (Terminal 2):**
```bash
curl http://localhost:5003/health
```

**Expected:** `{"status": "ok", "service": "rag-server"}`

**Step 3 - Deploy Docker services (Terminal 2):**
```bash
cd enrolment-app-open-ai
docker compose up --build -d
docker ps
```

**Verify 3 containers running:**
- frontend-service (port 8080)
- enrolment-service (port 5001)
- database-service (port 5002)

**Verify RAG server running locally:**
- rag-server (port 5003) - running in Terminal 1 (NOT containerized)

**Test RAG endpoints:**
```bash
# Direct RAG server
curl http://localhost:5003/health

# Via enrolment-service (which calls RAG at host.docker.internal:5003)
curl -X POST http://localhost:5001/rag/refresh
curl -X POST http://localhost:5001/rag/retrieve -d "query=students enrolled in ASD101&k=5"
curl -X POST http://localhost:5001/rag/answer -d "query=Which students are enrolled in ASD101?&k=5"
```

**Browser test:** `http://localhost:8080` → RAG tab

- Toggle RAG ON
- Click "Refresh Corpus" → verify `status: success`
- Enter query "students enrolled in ASD101" → Click "Retrieve Context" → verify results displayed
- Enter question "Which students are enrolled in ASD101?" → Click "Answer with Citations" → verify answer + citations + confidence

**Pass criteria:**
- RAG HTTP server running on port 5003 (Terminal 1)
- 3 Docker containers running (Terminal 2: docker ps)
- All curl commands return `status: success`
- RAG tab displays tool responses correctly
- Citations include source IDs
- Confidence category present (high/medium/low)

**Architecture Note:**
```
┌─────────────────────────────────────┐
│  Browser (localhost:8080)           │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  frontend-service (container)       │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  enrolment-service (container)      │
│  - Connects to database-service     │
│  - Connects to Ollama (host)        │
│  - Connects to RAG server (host)    │
└──────────────┬──────────────────────┘
               │
     ┌─────────┴──────────┬────────────┐
     ▼                    ▼            ▼
┌─────────┐    ┌──────────────┐   ┌──────────┐
│ database│    │ Ollama       │   │ RAG      │
│ (cont.) │    │ (host:11434) │   │ (host:   │
│         │    │              │   │  5003)   │
└─────────┘    └──────────────┘   └──────────┘
```

</details>

---

## 6. RAG Agentic Integration

<details>
<summary>What We're Doing</summary>

Add RAG mode to agentic_loop for automated validation of RAG pipeline implementation.

**Pattern:**
```
Manual Testing (Step 5) → Agentic Automation (Step 6) → Agentic Validation (Step 7)
```

**Components:**
* RAG collector - validates RAG server structure and 3 required tools
* RAG pipeline - formats evidence for dual-agent review
* Configuration - registers RAG mode
* Orchestrator - executes RAG validation workflow
* Menu - adds Option 6 (RAG)

</details>

<details>
<summary>RAG Validation Workflow</summary>

**Automated Evidence Collection:**

```
OBSERVE → ACT → ADAPT → RECORD
```

1. **Observe**: RAG collector scans rag-server/ for required files and tools
2. **Act**: Pipeline formats evidence for implementation agent (qwen2.5:0.5b)
3. **Adapt**: Review agent (llama3.1:8b) validates implementation quality using both review prompts
4. **Record**: Console output is copied into reports/rag-validation-report.md (agentic_loop prints results; it does not write files to disk)

**Validation Checks (rag_collector.py):**
* ✅ RAG server directory exists
* ✅ rag_pipeline.py, rag_server.py, and requirements.txt present
* ✅ 3 tools defined: refresh_corpus, retrieve_context, answer_question

</details>

<details>
<summary>RAG Collector Implementation</summary>

**File: agentic_loop/collectors/rag_collector.py**

```python
from pathlib import Path


REQUIRED_RAG_TOOLS = ["refresh_corpus", "retrieve_context", "answer_question"]


def collect(app_dir: Path, repo_root: Path) -> tuple[bool, str]:
    rag_server_dir = app_dir / "rag-server"

    required_paths = [
        rag_server_dir / "rag_pipeline.py",
        rag_server_dir / "rag_server.py",
        rag_server_dir / "requirements.txt",
    ]

    missing = [str(path.relative_to(app_dir)) for path in required_paths if not path.exists()]
    if missing:
        return False, "RAG evidence incomplete. Missing: " + ", ".join(missing)

    pipeline_text = (rag_server_dir / "rag_pipeline.py").read_text(encoding="utf-8")

    missing_tools = [tool for tool in REQUIRED_RAG_TOOLS if f"def {tool}" not in pipeline_text]
    if missing_tools:
        return False, "rag_pipeline.py missing required tools: " + ", ".join(missing_tools)

    return True, (
        "RAG evidence: rag-server/ contains rag_pipeline.py and rag_server.py; "
        f"{len(REQUIRED_RAG_TOOLS)} tools defined (refresh_corpus, retrieve_context, answer_question)."
    )
```

The collector receives `app_dir` and `repo_root` and returns `(bool, str)`, matching the calling convention every other collector (`db`, `endpoints`, `architecture`, `devops`, `mcp`) already uses in `core/orchestrator.py`.

</details>

<details>
<summary>RAG Pipeline Implementation</summary>

**File: agentic_loop/pipelines/rag_pipeline.py**

```python
def build_implementation_prompt(task_prompt: str, evidence: str) -> str:
    return f"""
{task_prompt}

Review Scope:
RAG Pipeline Integration

Observed Evidence:
{evidence}

Validate that:
1. All 3 RAG tools are defined and callable
2. refresh_corpus, retrieve_context, and answer_question follow their contracts
3. Answers are expected to include citations and a confidence category

Reply in at most 40 words and stay evidence-based.
""".strip()


def build_review_prompt(implementation_output: str, evidence: str) -> str:
    return f"""
Implementation Recommendation:
{implementation_output}

Observed Evidence:
{evidence}

Validate the RAG integration assessment against the evidence.
Identify any gaps or risks in retrieval quality or citation grounding.

Reply in at most 40 words and stay evidence-based.
""".strip()
```

Signatures mirror `mcp_pipeline.py`: `build_implementation_prompt(task_prompt, evidence)` and `build_review_prompt(implementation_output, evidence)`, both taking the string evidence returned by the collector.

</details>

<details>
<summary>Configuration Updates</summary>

**Update: agentic_loop/config/review_config.py**

Add the `"rag"` entry to the existing `build_mode_config()` function, after `"mcp"`:

```python
        "rag": ModeConfig(
            key="rag",
            label="RAG",
            prompt_family="lab8",
            implementation_prompts=("implementation/rag_implementation_prompt.txt",),
            review_prompts=("review/rag_review_prompt.txt", "review/rag_reasoning_prompt.txt"),
        ),
```

`build_mode_config()` now returns entries for `db`, `endpoints`, `architecture`, `devops`, `mcp`, and `rag`. Both `review_prompts` entries are loaded and combined by the orchestrator's `rag` branch (see Orchestrator Updates), so `rag_reasoning_prompt.txt` is wired in rather than left unused.

</details>


<details>
<summary>Orchestrator Updates</summary>

**Update: agentic_loop/core/orchestrator.py**

**Step 1:** Import `rag_collector` and `rag_pipeline` alongside the existing collectors/pipelines:

```python
from collectors import architecture_collector, db_collector, devops_collector, endpoints_collector, mcp_collector, rag_collector
from pipelines import architecture_pipeline, db_pipeline, devops_pipeline, endpoints_pipeline, mcp_pipeline, rag_pipeline
```

**Step 2:** Register `rag_collector.collect` in the `COLLECTORS` dict:

```python
COLLECTORS = {
    "db": db_collector.collect,
    "endpoints": endpoints_collector.collect,
    "architecture": architecture_collector.collect,
    "devops": devops_collector.collect,
    "mcp": mcp_collector.collect,
    "rag": rag_collector.collect,
}
```

**Step 3:** Add a `rag` branch to `run_mode()`, after the `mcp` branch, following the same evidence → implementation → review pattern:

```python
    if mode.key == "rag":
        _stage(mode.label, "PROMPTS", f"Loading prompt family: {mode.prompt_family}")
        task_prompt = prompts.read(mode.prompt_family, mode.implementation_prompts[0])
        system_prompt = (
            "You are a precise RAG pipeline validator. "
            "Use only supplied evidence and reply in at most 40 words."
        )
        implementation_user_prompt = rag_pipeline.build_implementation_prompt(task_prompt, evidence)
        _stage(mode.label, "PROMPTS", "Loaded RAG implementation prompt")

        _stage(mode.label, "LLM", "Running RAG implementation model")
        implementation_output, err = ai.call(system_prompt, implementation_user_prompt, review=False)
        if err:
            _stage(mode.label, "LLM", "Failed")
            return f"MODEL FAILED: {err}"
        _stage(mode.label, "LLM", "RAG implementation model complete")

        review_prompt_text = prompts.read(mode.prompt_family, mode.review_prompts[0])
        reasoning_prompt_text = prompts.read(mode.prompt_family, mode.review_prompts[1])
        review_system_prompt = f"{review_prompt_text}\n\n{reasoning_prompt_text}"
        review_user_prompt = rag_pipeline.build_review_prompt(implementation_output, evidence)
        _stage(mode.label, "PROMPTS", "Loaded RAG review and reasoning prompts")
        _stage(mode.label, "LLM", "Running RAG review model")
        review_output, review_err = ai.call(review_system_prompt, review_user_prompt, review=True)
        if review_err:
            review_output = review_err
            _stage(mode.label, "LLM", "Review model failed")
        else:
            _stage(mode.label, "LLM", "Review model complete")

        _stage(mode.label, "DONE", "Review complete")

        return (
            f"OBSERVE: {evidence}\n\n"
            f"IMPLEMENTATION: {implementation_output}\n"
            f"REVIEW: {review_output}"
        )
```

Both review prompts (`rag_review_prompt.txt` and `rag_reasoning_prompt.txt`) are concatenated into a single system prompt for the review model, so `rag_reasoning_prompt.txt` is no longer a dead asset — it now shapes the review call alongside the quality-check prompt.

This branch calls `collector(app_dir, repo_root)` and the pipeline functions with the same `(str, str)` signatures used by every other mode, which is why Fixes 1-2 (collector/pipeline signatures) were required for RAG mode to run at all.

</details>


<details>
<summary>Menu Updates: agentic_loop/app_main.py</summary>

**Step 1:** Add option `6` to `_menu_choice_to_key()`:

```python
def _menu_choice_to_key(choice: str) -> str | None:
    return {
        "1": "db",
        "2": "endpoints",
        "3": "architecture",
        "4": "devops",
        "5": "mcp",
        "6": "rag",
    }.get(choice)
```

**Step 2:** Add the `RAG` prompt path to `_print_mode_mapping()`:

```python
def _print_mode_mapping(app_dir: Path) -> None:
    prompt_map = {
        "DB": app_dir / "prompts" / "service",
        "Endpoints": app_dir / "prompts" / "service",
        "Architecture": app_dir / "prompts" / "lab4",
        "DevOps": app_dir / "prompts" / "lab5",
        "MCP": app_dir / "prompts" / "lab7",
        "RAG": app_dir / "prompts" / "lab8",
    }
    print_prompt_map({key: str(path) for key, path in prompt_map.items()})
```

**Step 3:** Add option `6 - RAG` to `print_menu()` in `agentic_loop/core/reporter.py`:

```python
def print_menu() -> None:
    print()
    print("=" * 70)
    print("AGENTIC REVIEW MENU")
    print("1 - DB")
    print("2 - Endpoints")
    print("3 - Architecture")
    print("4 - DevOps")
    print("5 - MCP")
    print("6 - RAG")
    print("0 - Exit")
    print("=" * 70)
```

</details>

---

## 7. Improvement Cycle

<details>
<summary>Workflow Overview</summary>

Execute RAG agentic validation using 3-terminal workflow.

**Pattern:**
```
Terminal A (Services) + Terminal B (RAG Server) + Terminal C (Agentic Loop)
```

**Process:**
1. **Observe**: Agentic_loop RAG collector scans implementation
2. **Act**: Dual-agent review validates RAG quality
3. **Adapt**: Review recommendations surface improvements
4. **Record**: Copy the printed OBSERVE/IMPLEMENTATION/REVIEW output into reports/rag-validation-report.md

</details>

<details>
<summary>Terminal A: Start Services</summary>

Start all microservices (database, enrolment, frontend):

```bash
cd enrolment-app-open-ai
docker-compose up
```

**Expected Output:**
```
database-service-1    | Server status: RUNNING on http://localhost:5002
enrolment-service-1   | Server status: RUNNING on http://localhost:5001
frontend-service-1    | Server status: RUNNING on http://localhost:8080
```

**Leave Terminal A running.**

</details>

<details>
<summary>Terminal B: Start RAG Server</summary>

Start standalone RAG server:

```bash
cd enrolment-app-open-ai/rag-server
python rag_server.py
```

**Expected Output:**
```
Starting Student Enrolment RAG MCP Server...
Server status: RUNNING
Interact with RAG tools from a second terminal.
Available tools:
- refresh_corpus
- retrieve_context
- answer_question
```

**Leave Terminal B running.**

</details>

<details>
<summary>Terminal C: Run Agentic Loop (RAG Mode)</summary>

Execute agentic_loop RAG validation:

```bash
cd enrolment-app-open-ai
python agentic_loop/agentic_loop.py
```

**Menu Selection:**
```
AGENTIC REVIEW MENU
1 - DB
2 - Endpoints
3 - Architecture
4 - DevOps
5 - MCP
6 - RAG
0 - Exit
Choose a review target: 6
```

**Expected Output:**

`agentic_loop.py` only prints stage lines and the final `OBSERVE`/`IMPLEMENTATION`/`REVIEW` text to the console — it does not write any report file.

```text
[RAG][START] Starting review flow
[RAG][OBSERVE] Collecting evidence
[RAG][OBSERVE] Complete
[RAG][PROMPTS] Loading prompt family: lab8
[RAG][PROMPTS] Loaded RAG implementation prompt
[RAG][LLM] Running RAG implementation model
[RAG][LLM] RAG implementation model complete
[RAG][PROMPTS] Loaded RAG review and reasoning prompts
[RAG][LLM] Running RAG review model
[RAG][LLM] Review model complete
[RAG][DONE] Review complete

RUNNING: RAG
OBSERVE: RAG evidence: rag-server/ contains rag_pipeline.py and rag_server.py; 3 tools defined (refresh_corpus, retrieve_context, answer_question).

IMPLEMENTATION: <qwen2.5:0.5b response - evidence-based, max 40 words>
REVIEW: <llama3.1:8b response - evidence-based, max 40 words>
```

- `OBSERVE` is the exact string returned by `rag_collector.collect()`.
- `IMPLEMENTATION` and `REVIEW` are live model output (qwen2.5:0.5b / llama3.1:8b), constrained to 40 words by the prompts — exact wording varies between runs.

**Record:** copy the printed `OBSERVE`/`IMPLEMENTATION`/`REVIEW` text above into `reports/rag-validation-report.md` yourself — the script does not save it for you.

</details>


<details>
<summary>Review Validation Report</summary>

`agentic_loop.py` does not write `reports/rag-validation-report.md` automatically. Create it from the Terminal C output:

```bash
mkdir -p enrolment-app-open-ai/reports
cat > enrolment-app-open-ai/reports/rag-validation-report.md << 'EOF'
# RAG Validation Report

## Evidence Collected (OBSERVE)

<paste the OBSERVE line printed by agentic_loop.py>

## Implementation Agent Assessment (qwen2.5:0.5b)

<paste the IMPLEMENTATION line>

## Review Agent Assessment (llama3.1:8b, review + reasoning prompts)

<paste the REVIEW line>
EOF
```

Replace each `<paste ...>` placeholder with the actual text your run produced, then save the file.

</details>

<details>
<summary>Improvement Cycle</summary>

**Observe (Evidence Collector):**
- ✅ RAG collector scanned rag-server/ directory
- ✅ Validated 3 required tools present
- ✅ Checked all required files exist

**Act (Implementation Agent):**
- ✅ qwen2.5:0.5b analyzed evidence
- ✅ Assessed RAG implementation correctness
- ✅ Verified Lab 08 requirements met

**Adapt (Review Agent):**
- ✅ llama3.1:8b reviewed implementation assessment using both review prompts
- ✅ Validated quality and confidence level
- ✅ Surfaced architecture-level risks and recommendations

**Record (Manual Report Creation):**
- ✅ OBSERVE/IMPLEMENTATION/REVIEW output copied into reports/rag-validation-report.md
- ✅ Evidence, assessments, and recommendations documented
- ✅ Metrics captured for improvement tracking

</details>

---

## 8. Evidence Log

<details>
<summary>Record Evidence</summary>

| Check | Expected Result | Actual Result | Pass/Fail |
|---|---|---|---|
| **RAG Server** |
| RAG server created | `rag-server/` exists with rag_pipeline.py, rag_server.py | | |
| 3 tools defined | refresh_corpus, retrieve_context, answer_question | | |
| **Terminal Testing** |
| RAG server starts | Terminal A shows "Server status: RUNNING" | | |
| Tools execute | Terminal B returns structured outputs | | |
| Evaluation runs | rag_eval.py outputs P@5, R@5 metrics | | |
| **UI Testing** |
| RAG HTTP server running | Terminal 1: `python3 rag_http_server.py` shows port 5003 | | |
| Docker services running | Terminal 2: docker ps shows 3 containers | | |
| RAG health check | curl http://localhost:5003/health returns ok | | |
| RAG endpoints work | curl commands return structured outputs | | |
| RAG UI functional | Browser displays RAG tab with tool buttons | | |
| **RAG Agentic Validation** |
| Collector created | `agentic_loop/collectors/rag_collector.py` exists | | |
| Pipeline created | `agentic_loop/pipelines/rag_pipeline.py` exists | | |
| Config updated | review_config.py includes "rag" mode | | |
| Orchestrator updated | orchestrator.py handles rag mode | | |
| Menu option added | agentic_loop.py shows Option 6 - RAG | | |
| Agentic loop runs | Terminal C shows RAG validation workflow | | |
| Evidence collected | OBSERVE output is the evidence string returned by rag_collector.py | | |
| LLM validation | IMPLEMENTATION and REVIEW outputs generated | | |
| Reports created | rag-validation-report.md | | |
| **Evidence** |
| rag-report.md created | Contains manual testing outputs | | |

**Required Evidence Files:**
- `reports/rag-report.md` - RAG manual testing outputs
- `reports/rag-validation-report.md` - Agentic validation results

</details>

---

## 9. Reflection

<details>
<summary>Answer Briefly:</summary>

1. What are the 3 RAG tools and their purposes?
2. How do retrieval metrics (P@5, R@5) validate answer quality?
3. What evidence did the agentic_loop RAG collector validate?
4. How did the dual-agent pattern improve RAG validation?

</details>

---

## 10. Key Learning Point

<details>
<summary>Learning Outcome</summary>

**RAG Integration Pattern:**

```
REFRESH → RETRIEVE → ANSWER → VALIDATE → REVIEW → IMPROVE
```

**Three Testing Layers:**
1. **Terminal** - RAG pipeline standalone tool execution
2. **UI** - Web interface RAG endpoints and controls
3. **Agentic** - Automated validation with dual-agent review

**Retrieval Quality = Safety:**
- P@5: Precision at 5 (relevant chunks in top 5)
- R@5: Recall at 5 (% of relevant chunks retrieved)
- Citations: Source traceability for all claims
- Confidence: Grounded in retrieval quality

**Automated Evidence Collection:**
- `rag_collector.py` validates all RAG artifacts exist
- `rag_pipeline.py` formats prompts with retrieval evidence
- Qwen 2.5 analyzes retrieval quality and answer grounding
- Llama 3.1 identifies citation gaps and confidence risks
- Reports document metrics and improvements

</details>
