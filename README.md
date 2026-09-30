# Multimodal RAG Pipeline

> Notebook-based retrieval pipeline for PDFs containing text, tables, and images.

![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)
![RAG](https://img.shields.io/badge/RAG-multimodal-8A2BE2)
![Chroma](https://img.shields.io/badge/Vector%20DB-Chroma-FF6F61)
![OpenAI](https://img.shields.io/badge/LLM-GPT--4o-412991?logo=openai&logoColor=white)
![Status](https://img.shields.io/badge/status-research%20prototype-orange)

## Overview

This repository explores a multimodal Retrieval-Augmented Generation pipeline for complex PDFs. Instead of flattening a document into plain text, the notebook preserves table HTML and embedded images, creates searchable AI-enhanced descriptions for mixed-content chunks, stores those descriptions in Chroma, and sends original text/tables/images back to a vision-capable model during answer generation.

The current project is intentionally a **research notebook / prototype**, not a production API.

## Pipeline

```text
PDF
 |
 v
Unstructured hi_res partitioning
 |-- text
 |-- tables -> HTML
 |-- images -> base64 payloads
 |
 v
Title-aware chunking
 |
 v
Mixed-content inspection
 |
 +--> raw text only --------------------+
 |                                       |
 +--> table/image chunk -> GPT-4o summary|
                                         v
                               LangChain Documents
                                         |
                                         v
                       OpenAI text-embedding-3-small
                                         |
                                         v
                                  Chroma vector DB
                                         |
                                         v
                                  top-k retrieval
                                         |
                                         v
                       original text + tables + images
                                         |
                                         v
                                   GPT-4o answer
```

## What the notebook implements

- High-resolution PDF partitioning with `unstructured.partition.pdf.partition_pdf`
- Table-structure inference with HTML preservation
- Image extraction as base64 payloads
- Title-aware chunking through `chunk_by_title`
- Configurable chunk boundaries and small-chunk combination
- Separation of text, table, and image content from original elements
- GPT-4o-generated searchable descriptions for multimodal chunks
- Fallback summaries if multimodal enrichment fails
- LangChain `Document` conversion with original multimodal content in metadata
- JSON export of processed chunks / retrieval results
- OpenAI `text-embedding-3-small` embeddings
- Persistent Chroma collections using cosine distance
- Top-k retrieval
- Multimodal final answer generation using retrieved raw text, table HTML, and images

## Example document

The repository includes the *Attention Is All You Need* paper under `docs/` as a reproducible sample for the notebook.

## Requirements

Python packages are listed in `requirements.txt`.

The Unstructured high-resolution PDF path also depends on native tools such as:

- Poppler / `poppler-utils`
- Tesseract OCR
- `libmagic`

You also need an OpenAI API key for embeddings and GPT-4o calls.

```env
OPENAI_API_KEY=your_key_here
```

## Run

Create a virtual environment, install the Python dependencies and native PDF/OCR dependencies, then open:

```text
multi_modal_rag.ipynb
```

Run the notebook from top to bottom. The demonstrated pipeline persists Chroma data under local `dbv*/chroma_db` directories.

## Retrieval design

A key design choice is separating **retrieval representation** from **answer-generation evidence**:

1. Mixed text/table/image chunks receive a searchable natural-language description.
2. That description is embedded for retrieval.
3. Original raw text, table HTML, and image payloads stay attached as metadata.
4. Retrieved original evidence is reconstructed for the final multimodal LLM call.

This makes semantically difficult visual/table content easier to retrieve without discarding the original evidence.

## Current limitations

- Notebook-oriented execution rather than reusable package modules
- Synchronous, sequential multimodal summarization
- No automated evaluation suite
- No API/service layer
- No ingestion job queue
- No document-level metadata filtering or tenancy
- API cost scales with multimodal chunk enrichment
- The committed notebook contains exploratory outputs that should be cleaned before productionizing

## Productionization roadmap

- [ ] Extract ingestion/retrieval/answering into tested Python modules
- [ ] Add configuration objects instead of notebook globals
- [ ] Add batch/concurrent enrichment with rate-limit control
- [ ] Add document metadata, namespaces, and tenant isolation
- [ ] Add retrieval/evaluation datasets and quality metrics
- [ ] Add caching and duplicate-content detection
- [ ] Add FastAPI ingestion/query endpoints
- [ ] Add background workers for expensive PDF processing
- [ ] Add observability, cost tracking, and CI
- [ ] Remove generated notebook output from the clean production path

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Keep experiments reproducible and describe retrieval-quality tradeoffs in pull requests.

## Status

Research/portfolio prototype demonstrating the end-to-end mechanics of multimodal document ingestion, retrieval, and grounded answer generation.
