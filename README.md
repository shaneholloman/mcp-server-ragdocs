# MCP-server-ragdocs
[![Node.js Package](https://github.com/sanderkooger/mcp-server-ragdocs/actions/workflows/release.yml/badge.svg)](https://github.com/sanderkooger/mcp-server-ragdocs/actions/workflows/release.yml)
![NPM Downloads](https://img.shields.io/npm/dy/%40sanderkooger%2Fmcp-server-ragdocs)
[![Version](https://img.shields.io/npm/v/@sanderkooger/mcp-server-ragdocs)](https://npmjs.com/package/@sanderkooger/mcp-server-ragdocs)
[![codecov](https://codecov.io/gh/sanderkooger/mcp-server-ragdocs/branch/main/graph/badge.svg)](https://codecov.io/gh/sanderkooger/mcp-server-ragdocs)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An MCP server implementation that provides tools for retrieving and processing documentation through vector search, enabling AI assistants to augment their responses with relevant documentation context.

## Table of Contents

- [Usage](#usage)
- [Features](#features)
- [Configuration](#configuration)
- [Deployment](#deployment)
  - [Local Development](#local-development)
  - [Cloud Deployment](#cloud-deployment)
- [Playwright Integration](#playwright-integration)
- [Tools](#tools)
- [Project Structure](#project-structure)
- [Using Ollama Embeddings](#using-ollama-embeddings)
- [License](#license)
- [Development Workflow](#development-workflow)
- [Contributing](#contributing)
- [Forkception Acknowledgments](#forkception-acknowledgments)

## Usage

The RAG Documentation tool is designed for:

- Enhancing AI responses with relevant documentation
- Building documentation-aware AI assistants
- Creating context-aware tooling for developers
- Implementing semantic documentation search
- Augmenting existing knowledge bases

## Features

- Vector-based documentation search and retrieval
- Support for multiple documentation sources
- Support for local (Ollama) embeddings generation or OPENAI
- Semantic search capabilities
- Automated documentation processing
- Real-time context augmentation for LLMs

## Configuration

```json
{
  "mcpServers": {
    "rag-docs": {
      "command": "npx",
      "args": ["-y", "@sanderkooger/mcp-server-ragdocs"],
      "env": {
        "EMBEDDINGS_PROVIDER": "ollama",
        "QDRANT_URL": "your-qdrant-url",
        "QDRANT_API_KEY": "your-qdrant-key" # if applicable
      }
    }
  }
}
```

### Usage with Claude Desktop

Add this to your `claude_desktop_config.json`:

### OpenAI Configuration

```json
{
  "mcpServers": {
    "rag-docs-openai": {
      "command": "npx",
      "args": ["-y", "@sanderkooger/mcp-server-ragdocs"],
      "env": {
        "EMBEDDINGS_PROVIDER": "openai",
        "OPENAI_API_KEY": "your-openai-key-here",
        "QDRANT_URL": "your-qdrant-url",
        "QDRANT_API_KEY": "your-qdrant-key"
      }
    }
  }
}
```

### Ollama Configuration

```json
{
  "mcpServers": {
    "rag-docs-ollama": {
      "command": "npx",
      "args": ["-y", "@sanderkooger/mcp-server-ragdocs"],
      "env": {
        "EMBEDDINGS_PROVIDER": "ollama",
        "OLLAMA_BASE_URL": "http://localhost:11434",
        "QDRANT_URL": "your-qdrant-url",
        "QDRANT_API_KEY": "your-qdrant-key"
      }
    }
  }
}
```

### Ollama run from this codebase
```
"ragdocs-mcp": {
      "command": "node",
      "args": [
        "/home/sander/code/mcp-server-ragdocs/build/index.js"
      ],
      "env": {
        "QDRANT_URL": "http://127.0.0.1:6333",
        "EMBEDDINGS_PROVIDER": "ollama",
        "OLLAMA_URL": "http://localhost:11434"
      },
      "alwaysAllow": [
        "run_queue",
        "list_queue",
        "list_sources",
        "search_documentation",
        "clear_queue",
        "remove_documentation",
        "extract_urls"
      ],
      "timeout": 3600
    }
```



## Environment Variables Reference

| Variable                | Required For  | Default                  | remarks                       |
|-------------------------|---------------|--------------------------|-------------------------------|
| `EMBEDDINGS_PROVIDER`   | All           | `ollama`                 | "openai" or "ollama"          |
| `OPENAI_API_KEY`        | OpenAI        | -                        | Obtain from OpenAI dashboard  |
| `OLLAMA_BASE_URL`       | Ollama        | `http://localhost:11434` | Local Ollama server URL       |
| `QDRANT_URL`            | All           | `http://localhost:6333`  | Qdrant endpoint URL           |
| `QDRANT_API_KEY`        | Cloud Qdrant  | -                        | From Qdrant Cloud console     |
| `PLAYWRIGHT_WS_ENDPOINT`| Playwright Remote | -                      | WebSocket endpoint for remote Playwright server (e.g., `ws://localhost:3000/`) |


### Local Deployment

The repository includes Docker Compose configuration for local development:

[Docker Compose Download](https://raw.githubusercontent.com/sanderkooger/mcp-server-ragdocs/main/docker-compose.yml)

```bash
docker compose up -d
```

This starts:

- Qdrant vector database on port 6333
- Ollama LLM service on port 11434

Access endpoints:

- Qdrant: http://localhost:6333
- Ollama: http://localhost:11434

### Cloud Deployment

For production deployments:

1. Use hosted Qdrant Cloud service
2. Set these environment variables:

```bash
QDRANT_URL=your-cloud-cluster-url
QDRANT_API_KEY=your-cloud-api-key
```

## Playwright Integration

This project supports running Playwright either locally or via a Docker container. This provides flexibility for environments where Playwright's dependencies might be challenging to install directly.

### How it Works

The `src/api-client.ts` file automatically detects the presence of the `PLAYWRIGHT_WS_ENDPOINT` environment variable:

- **If `PLAYWRIGHT_WS_ENDPOINT` is set**: The application will attempt to connect to a remote Playwright server at the specified WebSocket endpoint using `chromium.connect()`. This is ideal for using a containerized Playwright instance.
- **If `PLAYWRIGHT_WS_ENDPOINT` is not set**: The application will launch a local Playwright browser instance using `chromium.launch()`.

### Running Playwright in Docker

A `playwright` service has been added to the `docker-compose.yml` file to facilitate running Playwright in a Docker container.

To start the Playwright server in Docker:

```bash
docker-compose up playwright
```

This command will pull the `mcr.microsoft.com/playwright:v1.53.0-noble` image and start a Playwright server accessible on port `3000` of your host machine.

To configure your application to use this containerized Playwright instance, set the following environment variable:

```bash
PLAYWRIGHT_WS_ENDPOINT=ws://localhost:3000/
```

## Tools

### search_documentation

Search through stored documentation using natural language queries. Returns matching excerpts with context, ranked by relevance.

**Inputs:**

- `query` (string): The text to search for in the documentation. Can be a natural language query, specific terms, or code snippets.
- `limit` (number, optional): Maximum number of results to return (1-20, default: 5). Higher limits provide more comprehensive results but may take longer to process.

### list_sources

List all documentation sources currently stored in the system. Returns a comprehensive list of all indexed documentation including source URLs, titles, and last update times. Use this to understand what documentation is available for searching or to verify if specific sources have been indexed.

### extract_urls

Extract and analyze all URLs from a given web page. This tool crawls the specified webpage, identifies all hyperlinks, and optionally adds them to the processing queue.

**Inputs:**

- `url` (string): The complete URL of the webpage to analyze (must include protocol, e.g., https://). The page must be publicly accessible.
- `add_to_queue` (boolean, optional): If true, automatically add extracted URLs to the processing queue for later indexing. Use with caution on large sites to avoid excessive queuing.

### remove_documentation

Remove specific documentation sources from the system by their URLs. The removal is permanent and will affect future search results.

**Inputs:**

- `urls` (string[]): Array of URLs to remove from the database. Each URL must exactly match the URL used when the documentation was added.

### list_queue

List all URLs currently waiting in the documentation processing queue. Shows pending documentation sources that will be processed when run_queue is called. Use this to monitor queue status, verify URLs were added correctly, or check processing backlog.

### run_queue

Process and index all URLs currently in the documentation queue. Each URL is processed sequentially, with proper error handling and retry logic. Progress updates are provided as processing occurs. Long-running operations will process until the queue is empty or an unrecoverable error occurs.

### clear_queue

Remove all pending URLs from the documentation processing queue. Use this to reset the queue when you want to start fresh, remove unwanted URLs, or cancel pending processing. This operation is immediate and permanent - URLs will need to be re-added if you want to process them later.

## Project Structure

The package follows a modular architecture with clear separation between core components and MCP protocol handlers. See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed structural documentation and design decisions.

## Using Ollama Embeddings without docker

1. Install Ollama:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

2. Download the nomic-embed-text model:

```bash
ollama pull nomic-embed-text
```

3. Verify installation:

```bash
ollama list
```

## License

This MCP server is licensed under the MIT License. This means you are free to use, modify, and distribute the software, subject to the terms and conditions of the MIT License. For more details, please see the LICENSE file in the project repository.

## Contributing

We welcome contributions! Please see our [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines, but here are the basics:

1. Fork the repository
2. Install dependencies: `npm install`
3. Create a feature branch: `git checkout -b feat/your-feature`
4. Commit changes with npm run commit to ensure compliance with [Conventional Commits](https://www.conventionalcommits.org)
5. Push to your fork and open a PR

## Forkception Acknowledgments

This project is based on a fork of [hannesrudolph/mcp-ragdocs](https://github.com/hannesrudolph/mcp-ragdocs), which itself was forked from the original work by [qpd-v/mcp-ragdocs](https://github.com/qpd-v/mcp-ragdocs). The original project provided the foundation for this implementation.
