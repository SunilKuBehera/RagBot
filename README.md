# RagBot

A small PDF chatbot. It searches your documents first, then asks a language model to answer only from those passages.

The flow follows a plain RAG pipeline:

1. **Chunk** each PDF into short overlapping passages.
2. **Embed** those passages on your machine and store them in Chroma.
3. **Retrieve** the passages closest to the question.
4. **Generate** a grounded answer through Groq's free API.

Embeddings use Chroma's local `all-MiniLM-L6-v2` model, so document text is not sent to an embedding provider. Only the question and the retrieved passages go to Groq. The chat module calls them with the OpenAI Python SDK and `GROQ_BASE_URL=https://api.groq.com/openai/v1`. Follow-up questions are rewritten into a standalone search query before retrieval, and the chat shows the passages that were used.

## Stack

| Piece | Library |
| --- | --- |
| PDF text | [pypdf](https://pypdf.readthedocs.io/) |
| Vector store and local embeddings | [Chroma](https://www.trychroma.com/) |
| Chat client | [OpenAI SDK](https://github.com/openai/openai-python), pointed at Groq |
| Answers | [Groq](https://console.groq.com/) free API |
| Chat window | [Streamlit](https://streamlit.io/) |

## Setup

Python 3.10 or newer.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env
```

Open `.env` and set `GROQ_API_KEY`. Create a free key at [console.groq.com/keys](https://console.groq.com/keys).

## Layout

```
RAGBOT/
  app.py                 Streamlit chat
  docs/                  PDFs (a sample ships with the repo)
  eval/questions.json    five labeled retrieval checks
  ragbot/
    config.py            paths and settings from .env
    ingest.py            load, clean, and chunk PDFs
    store.py             build the Chroma collection
    retriever.py         nearest passages
    chat.py              query rewrite and grounded answer
    eval_retrieval.py    retrieval hit rate
  chroma_db/             local index (not committed)
  logs/                  question log (not committed)
```

The pipeline modules and the chat window are added in later commits. This commit is the project skeleton: license, ignore rules, dependencies, and the env template.

## Run

Once `app.py` is in the repo:

```powershell
streamlit run app.py
```

Put extra PDFs in `docs/`, then use **Rebuild index** in the sidebar. The first launch embeds the documents and saves them under `chroma_db/`. Later launches reuse that index.

Check retrieval on the sample questions:

```powershell
python -m ragbot.eval_retrieval
```

## License

MIT. See [LICENSE](LICENSE).
