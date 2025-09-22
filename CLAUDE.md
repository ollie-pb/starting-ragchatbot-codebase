# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install/sync dependencies
uv sync

# Add new dependency
uv add package-name
```

### Environment Setup
- Copy `.env.example` to `.env` and add your `ANTHROPIC_API_KEY`
- Application runs on `http://localhost:8000`
- API docs available at `http://localhost:8000/docs`

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for course materials with a tool-based search approach using Anthropic's Claude AI.

### Core Components Pipeline

1. **Document Processing** (`document_processor.py`)
   - Parses structured course documents with metadata (title, instructor, lessons)
   - Implements sentence-based chunking with configurable overlap
   - Extracts lesson markers (`Lesson N: Title`) and preserves hierarchical structure

2. **Vector Storage** (`vector_store.py`)
   - Uses ChromaDB with SentenceTransformer embeddings (`all-MiniLM-L6-v2`)
   - Stores course metadata and content chunks separately
   - Maintains course/lesson context in chunk metadata

3. **AI Generation** (`ai_generator.py`)
   - Integrates with Anthropic Claude (`claude-sonnet-4-20250514`)
   - Tool-based approach: AI decides when to search based on queries
   - Handles conversation context and tool execution

4. **Search Tools** (`search_tools.py`)
   - `CourseSearchTool` performs semantic vector similarity search
   - Returns relevant chunks with source metadata
   - Manages search result sources for response attribution

5. **Session Management** (`session_manager.py`)
   - Maintains conversation history per session
   - Configurable history length (`MAX_HISTORY = 2`)

### Request Flow

**Frontend → Backend:**
- User query sent to `/api/query` endpoint
- FastAPI validates request and creates/retrieves session
- `RAGSystem.query()` orchestrates the pipeline

**RAG Processing:**
- Query wrapped with instructions for Claude
- Conversation history retrieved if session exists
- AI generates response using available search tools
- Tool manager executes searches when AI determines relevance
- Sources extracted and conversation history updated

### Key Configuration

All settings are centralized in `backend/config.py`:
- `CHUNK_SIZE = 800` - Text chunk size for embeddings
- `CHUNK_OVERLAP = 100` - Overlap between chunks for context
- `MAX_RESULTS = 5` - Vector search result limit
- `CHROMA_PATH = "./chroma_db"` - Vector database storage

### Data Structure

Course documents in `docs/` follow this format:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: Introduction
[content...]

Lesson 1: Topic Name
Lesson Link: [url]
[content...]
```

### Frontend Architecture

Vanilla JavaScript SPA (`frontend/`) with:
- Real-time chat interface with session persistence
- Course statistics display via `/api/courses` endpoint
- Loading states and error handling
- Suggested questions for user guidance