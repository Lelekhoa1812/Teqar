title Multi-Agent Document Assistant – Ingestion, Agentic RAG, and Outcome Writing

// ---------- Clients & Entry Points ----------
Clients [color: orange, icon: monitor] {
  Web UI (Client) [icon: monitor, color: orange, label:"Label root files/folders; trigger actions"]
  Local FS Root [icon: folder, color: orange, label:"User-chosen directory"]
  Cloud Drives [icon: cloud, color: orange, label:"Google Drive / OneDrive link"]
}

// ---------- Models ----------
Models [color: purple, icon: cpu] {
  Llama-3.1-8B-Instruct [icon: cpu, color: purple, label:" Llama: Plan/Action"]
  PaddleOCR-VL [icon: cpu, color: purple, label:"PaddleOCR: Tables/Charts"]
  DeepSeek-OCR [icon: cpu, color: purple, label:"DeepSeek-OCR: Markdown formatter"]
  EmbeddingGemma-300M [icon: cpu, color: purple, label:"EmbeddingGemma-300M 768-d"]
}

// ---------- Agents & Services ----------
Agents|Services [color: green, icon: server] {
  FastAPI Backend [icon: server, color: green, label:"API gateway · job orchestration"]
  Ingestion Planner [icon: workflow, color: green, label:"Ingestion Planner: Batch | On-change planning · retries"]
  File Watcher [icon: activity, color: green, label:"File Watcher: events · debounce · hash-diff"]
  Parser Hub [icon: package, color: green, label:"Parser · OCR fallback"]
  Normaliser [icon: sliders, color: green, label:"Clean · unit/term std · Markdown"]
  Semantic Chunker [icon: layout, color: green, label:"Heading/table aware · windowing"]
  Summariser [icon: message-square, color: green, label:"Summariser: Per-file/folder factual"]
  Retriever [icon: search, color: green, label:"Retriever: (HNSW | FAISS)"]
  Outcome Filler [icon: pen-tool, color: green, label:"Outcome Filler: Template"]
  Back-Search & Exhaust [icon: compass, color: green, label:"Back-Search & Exhaust"]
  CRUD Manager [icon: refresh-ccw, color: green, label:"Upsert/delete chunks · re-index"]
  Verifier [icon: check-circle, color: green, label:"Verifier: Source trace"]
}

// ---------- MCP/Tools ----------
Tools (MCP) [color: pink, icon: wrench] {
//   PDF Reader (MCP) [icon: file-text, color: pink, label:"PDF → images/text"]
  DOCX Reader&Writer (MCP) [icon: file-text, color: pink, label:"DOCX Reader&Writer (MCP)"]
  Excel Reader&Writer (MCP) [icon: table, color: pink, label:"Excel Reader&Writer (MCP)"]
//   Markdown Converter [icon: file-text, color: pink, label:"Structure-preserving"]
  Table Normaliser [icon: grid, color: pink, label:"Header fix · wide-table splits"]
  Full-Text Scanner [icon: search, color: pink, label:"Keyword | regex fallback"]
}

// ---------- Data Artifacts & Stores ----------
Data Stores [color: blue, icon: database] {
  Config Store (MongoDB) [icon: database, color: blue, label:"Root map · labels (Device/Price/Outcome)"]
  Raw Store (MongoDB|GridFS) [icon: database, color: blue, label:"Full Markdown per file"]
  Chunk Store (MongoDB) [icon: database, color: blue, label:"Chunk text + metadata"]
  Vector Index (Mongo HNSW) [icon: aws-machine-learning, color: blue, label:"HNSW Runtime Index"]
  FAISS Runtime Index [icon: aws-machine-learning, color: blue, label:"FAISS Runtime Index"]
  Summary Store (MongoDB) [icon: database, color: blue, label:"Per-file/folder key facts"]
  Logs & Metrics [icon: database, color: blue, label:"ETL + retrieval traces"]
}

// ---------- Infrastructure ----------
Infra [color: gray, icon: cloud] {
  AWS EC2 (GPU) [icon: cloud, color: gray, label:"Local CUDA or cloud GPU runtime"]
  Docker Runtime [icon: box, color: gray, label:"Isolated services"]
  Local CUDA Host (alt) [icon: hard-drive, color: gray, label:"On-prem GPU server"]
}

// ---------- Flows : Setup ----------
Web UI (Client) > FastAPI Backend: "Select root or link Drive"
FastAPI Backend > Config Store (MongoDB): "Persist mapping & labels"
Cloud Drives > FastAPI Backend: "OAuth link · file list"
Local FS Root > File Watcher: "Watch paths"

// ---------- Flows : Ingestion ----------
File Watcher > Ingestion Planner: "Create/update job (file added/changed)"
Ingestion Planner > Parser Hub: "Route by type (PDF DOCX XLSX)"
Parser Hub > PaddleOCR-VL: "Image/scan OCR"
Parser Hub > DeepSeek-OCR: "Markdown OCR (long pages)"
Parser Hub > DOCX Reader&Writer (MCP): "Extract text/tables"
Parser Hub > Excel Reader&Writer (MCP): "Read sheets → tables"
PaddleOCR-VL > Normaliser: "Text + layout"
DeepSeek-OCR > Normaliser: "Markdown (structure-first)"
DOCX Reader&Writer (MCP) > Normaliser: "Text + tables"
Excel Reader&Writer (MCP) > Table Normaliser: "Header/units"
Table Normaliser > Normaliser: "Clean, unify units/terms"
Normaliser > Raw Store (MongoDB|GridFS): "Full Markdown"
Normaliser > Semantic Chunker: "Section/table aware chunks"
Semantic Chunker > EmbeddingGemma-300M: "Batch embeddings"
EmbeddingGemma-300M > Vector Index (Mongo HNSW): "Upsert vectors"
Semantic Chunker > Chunk Store (MongoDB): "Upsert chunk text/meta"
Raw Store (MongoDB|GridFS) > Summariser: "Summarise file/folder"
Summariser > Summary Store (MongoDB): "Persist key facts"
CRUD Manager > Chunk Store (MongoDB): "Deletions|updates"
CRUD Manager > Vector Index (Mongo HNSW): "Deletions|updates"


// ---------- Flows : Agentic RAG (Query/Outcome) ----------
Web UI (Client) > FastAPI Backend: "Run: Fill Outcome / Answer query"
FastAPI Backend > Llama-3.1-8B-Instruct: "Task + constraints (accuracy-first · no fabrication)"
Llama-3.1-8B-Instruct > Retriever: "Generate searches (keywords/synonyms) + filters (Device/Pricing)"
Retriever > Vector Index (Mongo HNSW): "kNN top-k"
Retriever > FAISS Runtime Index: "kNN top-k"
Vector Index (Mongo HNSW) > Retriever: "Chunk refs + scores"
FAISS Runtime Index > Retriever: "Chunk refs + scores"
Retriever > Llama-3.1-8B-Instruct: "Top-k chunks (with metadata)"
Llama-3.1-8B-Instruct > Back-Search & Exhaust: "If low confidence: broaden search"
Back-Search & Exhaust > Full-Text Scanner: "Keyword/regex scan (global)"
Full-Text Scanner > Llama-3.1-8B-Instruct: "Raw hits/snippets"
Llama-3.1-8B-Instruct > Verifier: "Trace sources · check units/consistency"
Llama-3.1-8B-Instruct > Outcome Filler: "Slot values + business-friendly prose"
Outcome Filler > DOCX Reader&Writer (MCP): "Write placeholders/bookmarks"
Outcome Filler > Excel Reader&Writer (MCP): "Write cells/sheets"
DOCX Reader&Writer (MCP) > Local FS Root: "Save updated Outcome.docx"
Excel Reader&Writer (MCP) > Local FS Root: "Save updated Outcome.xlsx"
Llama-3.1-8B-Instruct > Web UI (Client): "Answer + source list"

// ---------- Flows : Missing Data Handling ----------
Llama-3.1-8B-Instruct > Outcome Filler: "Insert '(Not found in available documents)' if exhausted"
Outcome Filler > Logs & Metrics: "Gap logged (field, file, query)"

// ---------- Flows : Updates ----------
File Watcher > Ingestion Planner: "Detect modified/added/removed"
Ingestion Planner > CRUD Manager: "Re-embed, delete stale, refresh summaries"

// ---------- Governance (Internal) ----------
FastAPI Backend > Logs & Metrics: "ETL timing · retrieval latencies · fill coverage"
Models > Infra: "Pinned to GPU/CPU pools (no external APIs)"
