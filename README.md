# Agentic RAG demo

Two notebooks, same retrieval pipeline, different router.

| Notebook | What it is | Who picks the route |
| --- | --- | --- |
| [`001-agentic-router.ipynb`](001-agentic-router.ipynb) | Deep Dive Agentic Retrieval Augmented Generation | An OpenAI chat prompt. The model returns JSON with `action` and `reason`. |
| [`002-agentic-router-with-jev.ipynb`](002-agentic-router-with-jev.ipynb) | Agentic RAG with TypeSafe Jev | Jev (TypeSafe) via `TYPESAFE_API_KEY`. A typed Choice, no router prompt. OpenAI writes the answer after retrieval. For a compound question it also splits the query and writes the combined answer. Jev still picks each route. |

Both send the chosen route to the same three tools:

- `OPENAI_QUERY` — Qdrant collection `opnai_data`
- `10K_DOCUMENT_QUERY` — Qdrant collection `10k_data` (Uber 2021 and Lyft 2024 10-K filings)
- `INTERNET_QUERY` — live Google search through SerpApi

## Read them

On GitHub, open the `.ipynb` file. The rendered page is the notebook.

- [001 on GitHub](https://github.com/Muthukumar-Selvarasu/agentic-RAG-demo/blob/main/001-agentic-router.ipynb)
- [002 on GitHub](https://github.com/Muthukumar-Selvarasu/agentic-RAG-demo/blob/main/002-agentic-router-with-jev.ipynb)

Use the **Open in Colab** badge at the top of the file you are reading. Each badge opens that file.

- `001` opens `001-agentic-router.ipynb` from this repo.
- `002` opens `002-agentic-router-with-jev.ipynb` from this repo.

The badge in `002` used to point at the `001` gist. If Colab shows the title **Deep Dive Agentic Retrieval Augmented Generation**, you are in `001`. The Jev notebook title is **Agentic RAG with TypeSafe Jev**.

## Setup

```bash
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip
pip install \
  torch==2.6.0 \
  transformers==4.48.0 \
  openai==1.59.9 \
  qdrant-client==1.13.3 \
  python-dotenv==1.0.1 \
  nest-asyncio==1.6.0 \
  ipykernel==6.29.5 \
  matplotlib==3.10.0 \
  einops==0.8.0 \
  typesafe-sdk
```

`typesafe-sdk` is only required for `002`. It needs Python 3.10 or newer. The notebook install cell also installs it.

```bash
cp .env.example .env
```

| Variable | `001` | `002` |
| --- | --- | --- |
| `OPENAI_API_KEY` | Router and RAG answers | RAG answers. Also splits a compound question and writes the combined answer. |
| `SERPAPI_KEY` | `INTERNET_QUERY`. `SERP_API_KEY` also works. | Same |
| `TYPESAFE_API_KEY` | Not used | `route_query` |

## Test locally in Cursor

1. Open the notebook you want to test.
2. Kernel picker (top right) → **Select Another Kernel** → **Jupyter Kernel** → **Python 3.13 (agentic-RAG)**  
   (or **Python Environments** → `.venv`).
3. Run the cells from the top through the Qdrant client cell. On a machine without `Agentic_RAG/qdrant_data`, that download cell fetches `10k_data` and `opnai_data`. The folder is gitignored. Colab downloads the same snapshot into `/content/Agentic_RAG/qdrant_data`. The collection is not in this GitHub repo, so a Colab run that skips the download cell opens an empty database and returns `Collection 10k_data not found`.
4. Run one query for each route.

Do not keep the default **Python 3.9** kernel. That is macOS system Python. It has no `python-dotenv`, and it is what produced `No module named 'dotenv'` plus the LibreSSL warning.

`google.colab` exists only on Colab. Locally that import fails on purpose, and the next lines load `.env`.

If the 3.13 kernel is already selected, you can skip the pip install cell.

### Queries

Run `agentic_rag(...)` in whichever notebook is open.

| Query | Expected route |
| --- | --- |
| `agentic_rag("what was uber revenue in 2021?")` | `10K_DOCUMENT_QUERY` |
| `agentic_rag("how to work with chat completions?")` | `OPENAI_QUERY` |
| `agentic_rag("List me down new LLMs in 2025")` | `INTERNET_QUERY` |

Internet queries do not need the Qdrant folder. The other two do. `Collection 10k_data not found` means the client was opened before the snapshot existed. Re-run the Qdrant client cell, then run the query again.

### What you should see

In `001`, the router cell calls the OpenAI client and returns `action` plus a short `reason`.

In `002`, the router cell prints a line like this before the answer:

```text
Decision made by TypeSafe (Jev): 10K_DOCUMENT_QUERY (confidence 1.00, probability 1.00)
```

The answer text after that line still comes from OpenAI for the Qdrant routes, and from SerpApi snippets for `INTERNET_QUERY`.

`agentic_rag_multi()` in either notebook splits a compound question, routes each part on its own, and returns one answer with the citations kept. In `002`, each of those routes is still a Jev decision. The Part 1 test cell and the bonus self-check at the bottom of each notebook are the cells that print those results.

## Versions

| Piece | Version | Why |
| --- | --- | --- |
| Python | **3.13** | Compatible with the notebook's Hugging Face pin. Do not use the default `python3` (3.14) — `transformers==4.48.0` and matching PyTorch wheels target 3.13 and earlier. |
| transformers | **4.48.0** | Pinned in the notebook install cell. Required by `nomic-ai/nomic-embed-text-v1.5`. |
| torch | **2.6.0** | First PyTorch line that ships Python 3.13 wheels and satisfies transformers 4.48 (`torch>=2`). |
| openai | **1.59.9** | Notebooks use `from openai import OpenAI`. |
| qdrant-client | **1.13.3** | Local on-disk client (`AsyncQdrantClient(path=...)`). |
| typesafe-sdk | **0.7.x** | `002` only. `TypeSafeClient` reads `TYPESAFE_API_KEY`. |
| python-dotenv | **1.0.1** | Loads `.env` outside Colab. |
| nest-asyncio | **1.6.0** | Lets the notebook run Qdrant's async client inside Jupyter. |
| ipykernel | **6.29.5** | Cursor / VS Code notebook kernel. |
| matplotlib | **3.10.0** | Imported by the notebooks. |
| einops | **0.8.0** | Required by the Nomic embedding model (`trust_remote_code=True`). |

Embedding model, downloaded on the first retrieve cell: `nomic-ai/nomic-embed-text-v1.5`.
