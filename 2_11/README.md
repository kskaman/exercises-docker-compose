# Exercise 2.11 — Containerized Development Environment

## Project: SourceTrace

Repository: https://github.com/kskaman/sourceTrace

I used Docker Compose in my **SourceTrace** project, a Python command-line Retrieval-Augmented Generation (RAG) application. SourceTrace parses documents, divides them into chunks, generates embeddings, and stores searchable data in PostgreSQL using the pgvector extension. It uses Ollama for local language-model inference.

Instead of installing and configuring database servers directly on my Windows machine, I use the project's `docker-compose.yaml` to run its development dependencies in containers:

- **`pgvector`** runs PostgreSQL with the pgvector extension. It is the application database and publishes container port 5432 to `localhost:5432`.
- **`pgvector-test`** runs a separate PostgreSQL/pgvector instance, published on `localhost:5433`, so automated tests do not interfere with application data.
- **`qdrant`** provides an additional vector database service on `localhost:6333` for development and experimentation.

Each service uses a Docker named volume, so its data persists when the container is stopped or recreated. The Python CLI and Ollama run on the host; this is a **partially containerized development environment**, not a claim that every component runs in Docker.

## How I use it on Windows

From the SourceTrace repository root, I start the required services in PowerShell:

```powershell
docker compose up -d pgvector pgvector-test
```

To start all configured services, including Qdrant:

```powershell
docker compose up -d
```

I check their status using:

```powershell
docker compose ps
```

The application uses a local `.env` file copied from `.env.example`. Its database URL must match the credentials configured in Docker Compose, for example:

```env
DATABASE_URL=postgresql://rag-cli:rag-cli@localhost:5432/rag-cli
TEST_DATABASE_URL=postgresql://rag-cli:rag-cli@localhost:5433/sourceTraceTest
```

I can then run the Python application and tests from the host machine:

```powershell
python rag_cli.py list-folders
python -m pytest -q
```

When I am finished, I stop and remove the containers and network without deleting the named volumes:

```powershell
docker compose down
```

## What I learned

Docker Compose makes my development database setup repeatable and keeps it separate from software installed directly on Windows. I can start multiple supporting services with one command, keep development and test databases isolated, and preserve their data with named volumes. It also makes it easier to reproduce the same supporting environment on another computer with Docker installed.
