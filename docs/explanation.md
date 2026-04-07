AI Trends Explorer — System Workflow & Architecture
What It Is
A RAG-based AI research intelligence platform for consulting teams. It automatically aggregates signals from academic papers, news feeds, and uploaded PDFs, then synthesizes them into actionable briefings using LLM-powered analysis.

End-to-End Workflow
The system follows a five-stage pipeline: Ingest → Chunk & Embed → Store → Retrieve & Synthesize → Present.

1. Data Ingestion (3 sources)
Source	Technology	Purpose
arXiv papers	arxiv Python client	Fetches abstracts from 5 categories (cs.AI, cs.LG, cs.CL, stat.ML, q-bio.QM) — covers cutting-edge research
RSS news feeds	feedparser	Parses 5 curated feeds (VentureBeat AI, MIT Tech Review, The Gradient, Google Research, OpenAI News) — covers industry signals
PDF uploads	LlamaIndex PDFReader	User-uploaded reports, processed asynchronously via Inngest — covers proprietary/custom content
seed_db.py runs the initial ingestion (~75 papers + ~50 articles). PDFs go through pages/upload.py at runtime.

2. Chunking & Embedding
Chunking: LlamaIndex SentenceSplitter (chunk_size=1000, overlap=200) in data_loader.py — splits documents at semantic boundaries while preserving context across chunks
Embedding: OpenAI text-embedding-3-large (3072 dimensions) — converts text chunks into dense vectors for similarity search
Deduplication: Deterministic UUID v5 from source URLs prevents duplicate ingestion
3. Vector Storage
Qdrant (Docker container) via vector_db.py — a purpose-built vector database using COSINE similarity
Each vector carries rich metadata: text, title, URL, date, source_type, category
Persistent Docker volume ensures data survives restarts
4. Retrieval & Synthesis (the RAG core)
trend_engine.py is the intelligence layer with three modes:

Mode	How It Works
Trend Dashboard	Embeds 7 predefined topic queries → retrieves top-6 similar chunks per topic → GPT-4o-mini synthesizes a 3-4 sentence analysis per trend
Weekly Digest	Retrieves signals across top-4 topics → GPT-4o-mini generates a structured executive briefing (summary + trends + strategic implications)
Explorer Q&A	Embeds user question → Qdrant search + OpenAI Responses API with web_search_preview tool for live web augmentation → hybrid answer combining knowledge base + web
The Explorer uses a hybrid retrieval strategy: knowledge base results are prioritized, but live web search fills gaps — with a graceful fallback to standard completions if the Responses API fails.

5. Frontend Presentation
Streamlit multi-page app with 4 views:

streamlit_app.py — Home page with system overview and flow diagram
pages/dashboard.py — 7 trend cards in a 2-column grid + query history
pages/digest.py — Downloadable weekly narrative briefing
pages/explorer.py — Chat interface with source citations and KB/Web badges
pages/upload.py — PDF upload with real-time progress polling
Technology Map

┌─────────────────────────────────────────────────────────┐
│  FRONTEND          Streamlit (multi-page app)           │
│                    Plotly (charts), Custom CSS           │
├─────────────────────────────────────────────────────────┤
│  LLM LAYER         OpenAI GPT-4o-mini (synthesis)       │
│                    OpenAI Responses API (web search)     │
│                    OpenAI text-embedding-3-large         │
├─────────────────────────────────────────────────────────┤
│  VECTOR DB          Qdrant (Docker, COSINE, 3072-dim)   │
├─────────────────────────────────────────────────────────┤
│  ASYNC BACKEND      FastAPI + Inngest (PDF workflows)   │
├─────────────────────────────────────────────────────────┤
│  INGESTION          arxiv client, feedparser, LlamaIndex│
├─────────────────────────────────────────────────────────┤
│  TOOLING            UV (deps), Docker, RAGAS (eval)     │
└─────────────────────────────────────────────────────────┘
Why Each Technology Was Chosen
Qdrant — lightweight, Docker-native vector DB; easy local setup with persistent storage
OpenAI text-embedding-3-large — high-quality 3072-dim embeddings for accurate semantic search
GPT-4o-mini — cost-efficient LLM that balances quality and speed for synthesis tasks
Inngest — durable async task orchestration for PDF processing (retries, step functions, monitoring dashboard)
FastAPI — serves as the Inngest webhook endpoint; lightweight and async-native
Streamlit — rapid prototyping of data-rich UIs with built-in caching (@st.cache_data with 1hr TTL)
LlamaIndex — robust PDF parsing and semantic chunking with sentence-boundary awareness
RAGAS — quantitative RAG evaluation (achieved 94.3% faithfulness, 73.6% answer relevancy)
UV — fast Python dependency management (replaces pip/poetry)