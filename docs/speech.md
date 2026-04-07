"AI Trends Explorer — How It Works"

Our platform solves a simple problem: keeping consulting teams up to date with AI research is overwhelming. There are hundreds of papers and articles published every week. We built a system that automatically collects, understands, and synthesizes those signals into actionable intelligence.

Here's how it works in five steps.

First, ingestion. We pull data from three sources: academic papers from arXiv across five AI categories, news articles from five curated RSS feeds like MIT Tech Review and VentureBeat, and any PDF report a user uploads directly. That gives us roughly a hundred signals to work with out of the box.

Second, chunking and embedding. Each document gets split into meaningful chunks using LlamaIndex's sentence splitter — this preserves context across boundaries. Then OpenAI's embedding model converts each chunk into a 3072-dimensional vector — essentially a numerical fingerprint of its meaning.

Third, storage. Those vectors go into Qdrant, a vector database running in Docker. When we need to find relevant content later, we compare vectors using cosine similarity — it's like asking "which chunks are closest in meaning to this query?"

Fourth, the intelligence layer — this is the core. When a user asks a question or views the dashboard, we embed the query, retrieve the most relevant chunks from Qdrant, and pass them as context to GPT-4o-mini. The model synthesizes an answer grounded in our actual data — that's the "retrieval-augmented generation" part. In the Explorer, we also augment with live web search for the freshest results.

Fifth, the interface. We built four views in Streamlit: a dashboard with seven trend cards, a weekly executive digest you can download, a chat-based explorer with source citations, and a PDF upload page with real-time processing status.

On the backend, FastAPI and Inngest handle the async PDF processing — so the UI never blocks while documents are being ingested.

We validated the system with RAGAS, a RAG evaluation framework, and scored 94% on faithfulness — meaning the answers stay grounded in the retrieved sources.

In short: we automated the research analyst's workflow — collect, read, synthesize, brief — and made it available through a clean interface that any consulting team can use.