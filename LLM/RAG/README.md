# LNG, Crude Oil, and Gold RAG with LangChain

This repository provides a Retrieval-Augmented Generation (RAG) system for **LNG**, **Crude Oil**, and **Gold (XAUUSD)**. The RAG component stores fundamental information such as reports, policy updates, and infrastructure data. The system also integrates time series data including prices, production, inventories, and exports. While RAG manages structured fundamental knowledge, the broader goal is to support a large language model (LLM) agent that performs news and information sentiment analysis to generate trading decisions.

---

## Objectives

* Maintain structured repositories of fundamental information for LNG, Crude Oil, and Gold.
* Combine textual reports and quantitative time series to provide accurate context.
* Enable hybrid retrieval with traceable citations.
* Support integration into a larger sentiment-driven trading decision framework.

---

## System Architecture

```
User → Router → (A) LNG_RAG
                    (B) Crude_RAG
                    (C) Gold_RAG
                    (D) Quantitative_TS
        → Answer composer with citations

Text data: loaders → cleaning → chunking → embeddings → vector store
Numeric data: EIA/JODI/market data → ETL → DuckDB/Parquet/Postgres → TimeSeries Tool
```

**Components**

* **Loaders**: Web and PDF loaders for EIA, OPEC, IGU, JODI, and other reports.
* **Text Processing**: RecursiveCharacterTextSplitter (chunk size 800–1200, overlap 100–150).
* **Embeddings**: OpenAI `text-embedding-3-large` or local models (e.g., bge-m3, jina-v3).
* **Vector Stores**: FAISS (prototype) or Milvus/Weaviate (production).
* **Retrievers**: Hybrid BM25 and vector search with optional reranking.
* **Tools**: `get_series()` for quantitative queries; additional functions for aggregation.
* **Router**: Few-shot classification to select LNG\_RAG, Crude\_RAG, Gold\_RAG, Quantitative\_TS, or Mixed.

---

## Project Layout

```
./
├─ apps/
│  └─ api.py                 # FastAPI entrypoint
├─ rag/
│  ├─ loaders.py             # Document loaders and cleaners
│  ├─ index_text.py          # Index construction
│  ├─ retrievers.py          # Hybrid retrievers
│  ├─ chains.py              # RAG chains (LNG, Crude, Gold)
│  ├─ router.py              # RouterChain
│  ├─ tools_timeseries.py    # Quantitative series utilities
│  └─ eval/                  # Evaluation modules
├─ data/                     # Raw, curated, and processed data
├─ vectorstores/             # Vector store persistence
├─ scripts/                  # ETL and scheduling
├─ .env                      # Environment variables
└─ README.md
```

---

## Setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -U langchain langchain-community langchain-openai
pip install -U pydantic bs4 unstructured pymupdf tiktoken
pip install -U faiss-cpu duckdb polars pyarrow pandas
pip install -U fastapi uvicorn
# Optional rerankers
pip install -U FlagEmbedding cohere
```

`.env` configuration:

```env
OPENAI_API_KEY=...
EIA_API_KEY=...           # https://www.eia.gov/opendata/
COHERE_API_KEY=...        # optional
VECTOR_BACKEND=faiss      # or milvus|weaviate
```

---

## Data Sources

### LNG

* **Textual**: EIA Natural Gas Weekly, IGU World LNG Report, Energy Institute Statistical Review.
* **Time Series**: Henry Hub spot price (EIA), LNG exports (EIA), TTF futures (exchange), JKM futures (CME/ICE).

### Crude Oil

* **Textual**: EIA Weekly Petroleum Status Report, EIA This Week in Petroleum, OPEC Monthly Oil Market Report, JODI Oil.
* **Time Series**: US crude inventories, production, refinery runs (EIA), Brent and WTI futures (exchange).

### Gold

* **Textual**: World Gold Council publications, IMF commodity notes, central bank reserve reports.
* **Time Series**: Spot and futures prices (COMEX, LBMA), ETF holdings, central bank purchases (IMF/World Gold Council).

---

## ETL and Refresh Strategy

* Text ingestion: scheduled downloads of reports and web scraping.
* Time series ingestion: scheduled API calls (EIA, IMF, LBMA, exchange-delayed data).
* Storage: DuckDB/Parquet for numeric series; vector stores for text.
* Scheduling: cron or workflow orchestration (Airflow/GitHub Actions).

---

## Answer Composition

* Responses always cite data with numbers, dates, and sources.
* Textual responses contain references to documents with metadata.
* Mixed responses combine quantitative data with RAG-based narratives.

---

## Evaluation

* **RAGAS**: Faithfulness, context precision, answer relevance.
* **Quantitative QA**: Verification against weekly and monthly EIA/JODI datasets.
* Monitoring: retrieval hit rate, citation frequency, latency, and classification accuracy.

---

## Implementation Checklist

* [ ] Acquire required API keys.
* [ ] Ingest seed reports and build indexes for LNG, Crude Oil, and Gold.
* [ ] Implement hybrid retrievers.
* [ ] Define quantitative series mappings for each product.
* [ ] Implement routing and RAG chains.
* [ ] Establish evaluation suites.
* [ ] Automate refresh and monitoring.

---

## License and Attribution

* Public and official data sources include EIA, OPEC, JODI, IGU, Energy Institute, and World Gold Council. Use is subject to each provider’s terms.
* Exchange data (ICE, CME, LBMA) may be delayed or licensed; use as proxies for research purposes.

---

## References

* EIA Open Data: [https://www.eia.gov/opendata/](https://www.eia.gov/opendata/)
* OPEC MOMR: [https://momr.opec.org/](https://momr.opec.org/)
* JODI Oil: [https://www.jodidata.org/oil/](https://www.jodidata.org/oil/)
* IGU Reports: [https://www.igu.org/resources/](https://www.igu.org/resources/)
* Energy Institute Statistical Review: [https://www.energyinst.org/statistical-review](https://www.energyinst.org/statistical-review)
* World Gold Council: [https://www.gold.org/research](https://www.gold.org/research)
