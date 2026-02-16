# Mem0 REST API Server

Mem0 provides a REST API server (written using FastAPI). Users can perform all operations through REST endpoints. The API also includes OpenAPI documentation, accessible at `/docs` when the server is running.

## Features

- **Create memories:** Create memories based on messages for a user, agent, or run.
- **Retrieve memories:** Get all memories for a given user, agent, or run.
- **Search memories:** Search stored memories based on a query.
- **Update memories:** Update an existing memory.
- **Delete memories:** Delete a specific memory or all memories for a user, agent, or run.
- **Reset memories:** Reset all memories for a user, agent, or run.
- **OpenAPI Documentation:** Accessible via `/docs` endpoint.

## Running the server locally

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- [Ollama](https://ollama.com/) running on your host machine

### 1. Pull the required Ollama models

```bash
ollama pull llama3.1:latest
ollama pull nomic-embed-text:latest
```

### 2. Configure environment variables

```bash
cd server
cp .env.example .env
```

The defaults work out of the box with the docker-compose setup. Edit `.env` if you need to override anything:

| Variable | Default | Description |
|---|---|---|
| `OLLAMA_BASE_URL` | `http://host.docker.internal:11434` | Ollama API endpoint (reachable from inside Docker) |
| `LLM_MODEL` | `llama3.1:latest` | Ollama model for LLM operations |
| `EMBEDDER_MODEL` | `nomic-embed-text:latest` | Ollama model for embeddings |
| `EMBEDDING_MODEL_DIMS` | `768` | Embedding vector dimensions (must match the embedder model) |
| `POSTGRES_HOST` | `postgres` | PostgreSQL host |
| `POSTGRES_PORT` | `5432` | PostgreSQL port |
| `POSTGRES_DB` | `postgres` | PostgreSQL database name |
| `POSTGRES_USER` | `postgres` | PostgreSQL user |
| `POSTGRES_PASSWORD` | `postgres` | PostgreSQL password |
| `POSTGRES_COLLECTION_NAME` | `memories` | PGVector collection name |
| `NEO4J_URI` | `bolt://neo4j:7687` | Neo4j connection URI |
| `NEO4J_USERNAME` | `neo4j` | Neo4j username |
| `NEO4J_PASSWORD` | `mem0graph` | Neo4j password |

### 3. Start the services

```bash
docker compose up -d
```

This starts the following services:

| Service | Port | Description |
|---|---|---|
| `mem0` | `8888` | Mem0 REST API server |
| `postgres` (pgvector) | `8432` | Vector store |
| `neo4j` | `8474` (HTTP), `8687` (Bolt) | Graph store |
| `mem0-mcp` | `8080` | MCP server (bridges Mem0 to MCP-compatible clients) |

### 4. Verify

```bash
# API docs
curl http://localhost:8888/docs

# Health check — add a test memory
curl -X POST http://localhost:8888/memories \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "I like dark mode"}], "user_id": "test"}'
```

## Architecture

```
Host machine (Ollama)
  :11434
     |
     v
Docker Compose
  ├── mem0        (:8888)  ── FastAPI + mem0ai
  │     ├──> postgres/pgvector (:8432)  — vector store
  │     └──> neo4j (:8687)             — graph store
  └── mem0-mcp    (:8080)  ── MCP bridge → mem0 API
```

The server uses **Ollama** for both LLM and embedding operations, removing the need for any external API keys. Ollama runs on the host and is accessed from Docker via `host.docker.internal`.

## Further reading

For more details, see the [official docs](https://docs.mem0.ai/open-source/features/rest-api).
