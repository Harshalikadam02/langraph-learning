# langraph-learning

## Overview

This repository demonstrates a LangGraph workflow integrating LangChain (OpenAI), LangGraph, and checkpoint persistence (MongoDB + in-memory fallback). It covers interactive usage, tool invocation, and human-in-the-loop support.

- `app/main.py`: interactive chat loop with graph state and checkpoint support.
- `app/support.py`: tool-call inspection, human answer resolution, and graph resume.
- `app/graph.py`: graph construction with `StateGraph`, tool node, and conditional transitions.
- `app/docker-compose.yml`: MongoDB service configuration for persistence.

## Tech stack

- `langgraph` for stateful graph orchestration.
- `langchain` + `langchain_openai` for LLM binding (`gpt-4.1`).
- `dotenv` for `OPENAI_API_KEY` management.
- `MongoDBSaver` (via `langgraph.checkpoint.mongodb`) for durable checkpointing.
- `InMemorySaver` for local dev without DB.

## Setup

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

cat > .env <<'EOL'
OPENAI_API_KEY=sk-...
EOL

docker compose -f app/docker-compose.yml up -d

python3 -m app.main
python3 -m app.support
```

## `app/graph.py` behavior

1. `@tool()` defines `human_assistance_tool` returning structured data.
2. `llm = init_chat_model(model_provider='openai', model='gpt-4.1')` and `llm.bind_tools(...)`.
3. `StateGraph` defines input `State` with messages.
4. Graph edges:
   - `START -> chatbot`
   - `chatbot -> tools` conditionally via `tools_condition`
   - `tools -> chatbot` after tool execution
   - `chatbot -> END`
5. `create_chat_graph(checkpointer=...)` compiles graph with checkpointing.

## checkpointing strategy

- `MongoDBSaver.from_conn_string(MONGODB_URI)` writes graph state to MongoDB.
- `InMemorySaver()` can be used as an alternative for no persistence.
- `config = {'configurable': {'thread_id': '7'}}` identifies execution instance.

## `app/support.py` flow

- Loads graph state: `graph_with_mongo.get_state(config=config)`.
- Inspects last message for tool call `human_assistance_tool`.
- Prompts user resolution and resumes graph with `Command(resume={'data': ans})`.

## Secrets & security

- `.env` is ignored via `.gitignore`.
- `venv/`, `__pycache__/`, `.DS_Store` ignored.
- Do not commit OpenAI API tokens.

## GitHub

Pushed to: `https://github.com/Harshalikadam02/langraph-learning.git`.

## Quick checks

```bash
python3 -c 'from dotenv import load_dotenv; import os; load_dotenv(); print(bool(os.getenv("OPENAI_API_KEY")))'
python3 -m app.main
```

## LangGraph expertise highlighted

- Graph construction & flow control
- tool integration with `ToolNode`
- checkpoint persistence
- recovery and resume pattern
- human-in-the-loop tool resolution
