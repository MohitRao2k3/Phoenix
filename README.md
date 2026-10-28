# LangGraph PDF Chatbot (RAG + Tools)

A multi-utility chatbot built with **LangGraph** and **Streamlit**. Upload a PDF to a chat and ask questions about it, or let the assistant use tools such as live stock prices and a calculator. Every conversation is saved, so you can reopen past chats from the sidebar.

## Features

- **Chat with your PDF (RAG):** upload a PDF per conversation, it is split into chunks and indexed, and answers are drawn from the document.
- **Tool use:** the assistant decides when to call tools (for example stock price lookup and a calculator) and shows a status box while a tool runs.
- **Streaming replies:** responses appear token by token.
- **Multiple conversations:** each chat has its own thread ID, and past chats can be reopened from the sidebar.
- **Per-chat documents:** a PDF belongs to the thread it was uploaded in.

## Project structure

```
.
├── streamlit_rag_frontend.py   # Streamlit UI: sidebar, PDF upload, chat, streaming
├── backend_with_rag.py         # LangGraph graph, tools, PDF ingestion, checkpointer
└── README.md
```

The frontend imports these from the backend: `chatbot`, `ingest_pdf`, `retrieve_all_threads`, `thread_document_metadata`.

## How it works

1. **Upload:** `ingest_pdf` reads the PDF, splits it into chunks, and stores them for the current `thread_id`.
2. **Ask:** the Streamlit app sends your message to the LangGraph `chatbot` with that `thread_id`.
3. **Decide:** the model either answers directly, searches the document, or calls a tool.
4. **Loop:** tool results go back to the model, which then writes the final answer.
5. **Save:** a checkpointer stores the conversation per `thread_id`, so old chats can be reloaded.

## Requirements

- Python 3.11 or newer
- An API key for the LLM provider used in `backend_with_rag.py` (Google Gemini in my setup)
- Optional keys for the tools you enable (for example Alpha Vantage for stock prices)

## Setup

```bash
# 1. Clone the repo
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# 2. Create and activate a virtual environment
python -m venv myenv
myenv\Scripts\activate          # Windows
# source myenv/bin/activate     # macOS / Linux

# 3. Install dependencies
pip install streamlit langgraph langchain-core langchain-google-genai python-dotenv requests
# plus whatever backend_with_rag.py imports for PDF loading and vector search, for example:
# pip install langchain-community langchain-text-splitters pypdf faiss-cpu
```

> Tip: once everything works, run `pip freeze > requirements.txt` and trim it to the packages you actually use.

## Configuration

Create a `.env` file in the project folder (never commit it):

```
GOOGLE_API_KEY=your_google_api_key
ALPHAVANTAGE_API_KEY=your_alpha_vantage_key
```

Add any other keys your backend reads, and keep `.env` in `.gitignore`.

## Run

```bash
streamlit run streamlit_rag_frontend.py
```

Then open the URL shown in the terminal (usually http://localhost:8501).

## Usage

1. Click **New Chat** to start a conversation.
2. Optionally upload a PDF in the sidebar and wait for "PDF indexed".
3. Ask questions about the document, or try prompts like:
   - "Summarize the uploaded document."
   - "What is the stock price of Tesla, and what would 50 shares cost?"
4. Click a thread ID under **Past conversations** to reopen an old chat.

## Notes and limitations

- Answers are only as good as the retrieved passages, and a PDF with scanned images and no text may not index properly.
- Stock prices depend on a third-party API with rate limits, and the data may be delayed.
- Chat history is stored by the backend's checkpointer. If it uses in-memory storage, history is lost when the app restarts.
- Never commit API keys. If one is exposed, regenerate it.

## Tech stack

- [LangGraph](https://langchain-ai.github.io/langgraph/) for the agent graph and persistence
- [LangChain](https://python.langchain.com/) for messages, tools and model wrappers
- [Streamlit](https://streamlit.io/) for the UI
- Google Gemini as the LLM

## License

Add a license of your choice (for example MIT) before sharing the repo publicly.
