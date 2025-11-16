# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start: Setting Up on a New Device

### Prerequisites
```bash
# macOS
brew install protobuf node
npm install -g pnpm

# Verify installations
protoc --version
pnpm --version
node --version  # >= 18 required
```

### First-Time Setup
```bash
# 1. Clone repository (forked from ag-ui-protocol/ag-ui)
git clone https://github.com/kga245/ag-ui.git
cd ag-ui

# 2. Install all dependencies
pnpm install

# 3. Build all packages (required before running)
pnpm build

# 4. Set up API keys for integrations you want to use
# Copy .env.example files and add your keys:
cp integrations/langgraph/python/examples/.env.example integrations/langgraph/python/examples/.env
cp integrations/crew-ai/python/.env.example integrations/crew-ai/python/.env
cp integrations/adk-middleware/python/examples/.env.example integrations/adk-middleware/python/examples/.env

# Edit each .env file to add your API keys

# 5. (Optional) Install Python dependencies for integrations
cd integrations/langgraph/python/examples && poetry install && cd -
cd integrations/crew-ai/python && poetry install && cd -
cd integrations/adk-middleware/python && poetry install && cd -

# 6. Start the Dojo frontend
cd apps/dojo && pnpm dev
# Access at http://localhost:3000
```

### After Pulling Updates
```bash
# Re-install dependencies (in case package.json changed)
pnpm install

# Rebuild all packages
pnpm build
```

### Syncing with Upstream (Original ag-ui-protocol/ag-ui)
```bash
# Fetch latest from original project
git fetch upstream

# Merge upstream changes into your branch
git merge upstream/main

# Or rebase your changes on top of upstream
git rebase upstream/main

# Push to your fork
git push origin main
```

## Common Development Commands

### TypeScript SDK (Main Development)
```bash

# Install dependencies (using pnpm)
pnpm install

# Build all packages
pnpm build

# Run development mode
pnpm dev

# Run linting
pnpm lint


# Run type checking
pnpm check-types

# Run tests
pnpm test

# Format code
pnpm format

# Clean build artifacts
pnpm clean

# Full clean build
pnpm build:clean
```

### Python SDK
```bash
# Navigate to python-sdk directory
cd python-sdk

# Install dependencies (using poetry)
poetry install

# Run tests
python -m unittest discover tests

# Build distribution
poetry build
```

### Running Specific Integration Tests
```bash
# For TypeScript packages/integrations
cd packages/<package-name>
pnpm test

# For running a single test file
cd packages/<package-name>
pnpm test -- path/to/test.spec.ts
```

### AG-UI Dojo (Demo Viewer)
```bash
# Start dojo frontend only (uses built-in mock agents)
cd apps/dojo
pnpm dev
# Access at http://localhost:3000

# Start all backend servers + frontend (requires .env setup)
cd apps/dojo
pnpm run run-everything
```

**Important**: The Dojo frontend connects to backend agent servers. Without backends running, you'll see `ECONNREFUSED` errors.

**Quick start without API keys**: Select "middleware-starter" integration from the dropdown - this uses in-memory mock agents requiring no backend.

**For LLM-powered integrations**, create `.env` files with `OPENAI_API_KEY`:
```bash
# Example for LangGraph Python
cd integrations/langgraph/python/examples
cp .env.example .env
# Edit .env to add: OPENAI_API_KEY=your-key

# Start the backend server
python -m uvicorn server:app --port 2024
```

Backend URLs (configured in `apps/dojo/src/env.ts`):
- LangGraph Python: `http://localhost:2024`
- Mastra: `http://localhost:4111`
- Agno: `http://localhost:9001`
- CrewAI: `http://localhost:9002`
- PydanticAI: `http://localhost:9000`
- Spring AI: `http://localhost:8080`

## High-Level Architecture

AG-UI is an event-based protocol that standardizes agent-user interactions. The codebase is organized as a monorepo with the following structure:

### Core Protocol Architecture
- **Event-Driven Communication**: All agent-UI communication happens through typed events (BaseEvent and its subtypes)
- **Transport Agnostic**: Protocol supports SSE, WebSockets, HTTP binary, and custom transports
- **Observable Pattern**: Uses RxJS Observables for streaming agent responses

### Key Abstractions
1. **AbstractAgent**: Base class that all agents must implement with a `run(input: RunAgentInput) -> Observable<BaseEvent>` method
2. **HttpAgent**: Standard HTTP client supporting SSE and binary protocols for connecting to agent endpoints
3. **Event Types**: Lifecycle events (RUN_STARTED/FINISHED), message events (TEXT_MESSAGE_*), tool events (TOOL_CALL_*), and state management events (STATE_SNAPSHOT/DELTA)

### Repository Structure
- `/sdks/typescript/`: Main TypeScript implementation
  - `/packages/`: Core protocol packages (@ag-ui/core, @ag-ui/client, @ag-ui/encoder, @ag-ui/proto)
- `/integrations/`: Framework integrations (langgraph, mastra, crewai, etc.)
- `/apps/`: Example applications including the AG-UI Dojo demo viewer
- `/sdks/python/`: Python implementation of the protocol
- `/docs/`: Documentation site content

### Integration Pattern
Each framework integration follows a similar pattern:
1. Implements the AbstractAgent interface
2. Translates framework-specific events to AG-UI protocol events
3. Provides both TypeScript client and Python server implementations
4. Includes examples demonstrating key AG-UI features (agentic chat, generative UI, human-in-the-loop, etc.)

### State Management
- Uses STATE_SNAPSHOT for complete state representations
- Uses STATE_DELTA with JSON Patch (RFC 6902) for efficient incremental updates
- MESSAGES_SNAPSHOT provides conversation history

### Multiple Sequential Runs
- AG-UI supports multiple sequential runs in a single event stream
- Each run must complete (RUN_FINISHED) before a new run can start (RUN_STARTED)
- Messages accumulate across runs (e.g., messages from run1 + messages from run2)
- State continues to evolve across runs unless explicitly reset with STATE_SNAPSHOT
- Run-specific tracking (active messages, tool calls, steps) resets between runs

### Development Workflow
- Turbo is used for monorepo build orchestration
- Each package has independent versioning
- Integration tests demonstrate protocol compliance
- The AG-UI Dojo app showcases all protocol features with live examples

## AG-UI Dojo Feature Reference

### Available Demo Features

| Feature | Description | Key Capabilities |
|---------|-------------|------------------|
| **agentic_chat** | Chat with your Copilot and call frontend tools | Basic chat, tool calling, streaming |
| **backend_tool_rendering** | Render and stream backend tools to frontend | Agent state management, collaboration |
| **human_in_the_loop** | Plan tasks collaboratively with human approval | HITL patterns, interactivity |
| **agentic_generative_ui** | Long-running tasks with dynamic UI generation | Agent-driven UI, async tasks |
| **tool_based_generative_ui** | Tool-driven UI generation (e.g., Haiku generator) | Action-based generative UI |
| **shared_state** | Bidirectional state synchronization between agent and UI | STATE_SNAPSHOT/DELTA events |
| **predictive_state_updates** | Real-time collaborative document editing | Streaming state updates |
| **agentic_chat_reasoning** | Chat with reasoning/thinking traces | Extended thinking, reasoning chains |
| **subgraphs** | Multi-agent architectures with nested workflows | Agent orchestration, sub-agents |
| **a2a_chat** | Agent-to-Agent protocol communication | A2A middleware, multi-agent routing |
| **vnext_chat** | CopilotKit vNext-based chat | Next-gen CopilotKit features |

### Integration Feature Matrix

**Most Complete Implementations** (7-9 features):
- **LangGraph (Python/FastAPI/TypeScript)**: 8-9 features - Most comprehensive, includes subgraphs
- **Microsoft Agent Framework (.NET/Python)**: 7 features - Full protocol support
- **Server Starter (All Features)**: 8 features - Reference implementation
- **CrewAI**: 7 features - Multi-agent workflows

**Medium Feature Support** (5-6 features):
- **Pydantic AI**: 6 features - Python-native, clean API
- **Spring AI**: 5 features - Java/Kotlin ecosystem
- **LlamaIndex**: 5 features - RAG-focused agents
- **Google ADK**: 5 features - Google's agent framework

**Basic Implementations** (1-4 features):
- **Mastra**: 3-4 features - TypeScript-first framework
- **Agno**: 3 features - Lightweight agent framework
- **Middleware Starter**: 1 feature - In-memory mock (no backend needed)
- **Server Starter**: 1 feature - Basic HTTP server template
- **A2A (Direct/Middleware)**: 1-2 features - Agent-to-Agent protocol

### Key Differences Between Integrations

**By Language/Runtime:**
- **Python**: LangGraph, Pydantic AI, CrewAI, LlamaIndex, Agno, Microsoft Agent Framework (Python)
- **TypeScript/JavaScript**: LangGraph-TypeScript, Mastra, Server Starter
- **Java/Kotlin**: Spring AI
- **.NET/C#**: Microsoft Agent Framework (.NET)

**By Architecture Pattern:**
- **Graph-based workflows**: LangGraph (state machines, subgraphs)
- **Multi-agent orchestration**: CrewAI (crews/teams), A2A Middleware
- **RAG-focused**: LlamaIndex (document retrieval)
- **Type-safe Python**: Pydantic AI (type validation)
- **Framework starters**: Middleware Starter (mock), Server Starter (HTTP template)

**Unique Features:**
- **LangGraph**: Only integration with `subgraphs` and `agentic_chat_reasoning` (Python/FastAPI)
- **A2A**: Dedicated agent-to-agent communication protocol
- **Middleware Starter**: Runs entirely in-memory, no backend server required (great for testing)

## Running Integration Backends

### LangGraph (Python) - Port 2024
```bash
# 1. Edit .env file with your API keys
# Location: integrations/langgraph/python/examples/.env
# Required: OPENAI_API_KEY
# Optional: LANGSMITH_API_KEY (for tracing)

cd integrations/langgraph/python/examples
poetry install
poetry run langgraph dev --port 2024
```

### CrewAI (Python) - Port 9002
```bash
# 1. Edit .env file with your API keys
# Location: integrations/crew-ai/python/.env
# Required: OPENAI_API_KEY (uses litellm with model="openai/gpt-4o")

cd integrations/crew-ai/python
poetry install
PORT=9002 poetry run python -m ag_ui_crewai.dojo
```

### Google ADK (Python) - Port 8000
```bash
# 1. Edit .env file with your API keys
# Location: integrations/adk-middleware/python/examples/.env
# Required: GOOGLE_API_KEY (Gemini API)

cd integrations/adk-middleware/python
poetry install
cd examples
PORT=8000 poetry run python -m server
```

### API Key Requirements Summary

| Integration | Required API Key | Provider | Optional Keys |
|-------------|-----------------|----------|---------------|
| **LangGraph** | `OPENAI_API_KEY` | OpenAI | `LANGSMITH_API_KEY`, `TAVILY_API_KEY` |
| **CrewAI** | `OPENAI_API_KEY` | OpenAI (via litellm) | `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY`, `TAVILY_API_KEY` |
| **Google ADK** | `GOOGLE_API_KEY` | Google Gemini | `TAVILY_API_KEY`, `SERPER_API_KEY` |

**litellm Note**: CrewAI uses litellm which supports multiple providers. You can change `model="openai/gpt-4o"` to:
- `model="anthropic/claude-3-sonnet"` (requires `ANTHROPIC_API_KEY`)
- `model="gemini/gemini-pro"` (requires `GOOGLE_API_KEY`)
- `model="azure/gpt-4"` (requires `AZURE_API_KEY` + `AZURE_API_BASE`)
- Add to memory