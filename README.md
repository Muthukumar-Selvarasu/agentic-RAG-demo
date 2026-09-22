# Agentic RAG demo

Local runtime for [`001-agentic-router.ipynb`](001-agentic-router.ipynb). Open the notebook in Cursor, select the `.venv` kernel, and run cells.

## Chosen versions

| Piece | Version | Why |
| --- | --- | --- |
| Python | **3.13** | Already on this machine (`/opt/homebrew/bin/python3.13`). Compatible with the notebook's Hugging Face pin. Do not use the default `python3` (3.14) — `transformers==4.48.0` and matching PyTorch wheels target 3.13 and earlier. |
| transformers | **4.48.0** | Pinned in the notebook install cell. Required by `nomic-ai/nomic-embed-text-v1.5`. |
| torch | **2.6.0** | First PyTorch line that ships Python 3.13 wheels and satisfies transformers 4.48 (`torch>=2`). |
| openai | **1.59.9** | Notebook uses the current `OpenAI()` client (`from openai import OpenAI`). |
| qdrant-client | **1.13.3** | Local on-disk client (`AsyncQdrantClient(path=...)`) used for the 10-K and OpenAI-docs collections. |
| python-dotenv | **1.0.1** | Loads `.env` when the notebook is not running on Colab. |
| nest-asyncio | **1.6.0** | Lets the notebook run Qdrant's async client inside Jupyter. |
| ipykernel | **6.29.5** | Cursor / VS Code notebook kernel. |
| matplotlib | **3.10.0** | Imported by the notebook. |
| einops | **0.8.0** | Required by the Nomic embedding model (`trust_remote_code=True`). |

Embedding model (downloaded on first retrieve cell): `nomic-ai/nomic-embed-text-v1.5`.

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
  einops==0.8.0
```

Copy secrets and fill them in:

```bash
cp .env.example .env
```

| Variable | Used by |
| --- | --- |
| `OPENAI_API_KEY` | Router + RAG generation |
| `SERPAPI_KEY` | Live Google search (`INTERNET_QUERY`). `SERP_API_KEY` also works. |
| `TYPESAFE_API_KEY` | TypeSafe / Jev. Not read by this notebook yet. |

## Run from Cursor

1. Open `001-agentic-router.ipynb`.
2. Kernel picker (top right) → **Select Another Kernel** → **Jupyter Kernel** → **Python 3.13 (agentic-RAG)**  
   (or **Python Environments** → `.venv`).
3. Re-run cells top to bottom.

Do not keep the default **Python 3.9** kernel. That is macOS system Python. It has no `python-dotenv`, and it is what produced `No module named 'dotenv'` plus the LibreSSL warning.

`google.colab` is only on [Google Colab](https://colab.research.google.com/github/). Locally that import is supposed to fail; the next line loads `.env` instead.

The first install cell now also installs `python-dotenv`, `nest-asyncio`, and `einops` so Colab and Cursor both work. If the 3.13 kernel is already selected, you can skip that cell.

## Qdrant data

OpenAI-docs and 10-K routes need the prebuilt store at `Agentic_RAG/qdrant_data`. On Colab the notebook clones [hamzafarooq/multi-agent-course](https://github.com/hamzafarooq/multi-agent-course). Locally that clone is skipped, so copy the snapshot next to this notebook before those cells:

```text
Agentic_RAG/qdrant_data
```

Internet / SerpApi queries do not need that folder.
