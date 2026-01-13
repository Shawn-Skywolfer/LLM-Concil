# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**贤者议会 (Council of Sages)** is a single-file React application that simulates multi-agent AI debates. Users input topics, and the system recruits 3-5 AI personas who engage in multi-round discussions with moderator-driven consensus.

**Key characteristics:**
- Zero-build architecture - runs directly in a browser
- All code in `index-V8.HTML` (~3000+ lines)
- OpenAI-compatible API integration with streaming responses
- Chinese language interface

## Development Commands

**This project has no build system.** To run:
```bash
# Simply open in browser
start index-V8.HTML

# Or serve via static server
python -m http.server 8000
# then visit http://localhost:8000
```

**No package.json, no npm install, no build step.** All dependencies are CDN-hosted (React from esm.sh, Tailwind via CDN, Babel standalone).

## Architecture

### Single-File Structure
The entire application is in `index-V8.HTML` with:
- CDN imports via import map
- Embedded React components defined as functions
- Inline Babel transpilation (`type="text/babel"`)
- Tailwind CSS via CDN script

### Component Hierarchy
```
CouncilOfSages (root)
├── Sidebar (sessions, new topic, backup controls)
├── SettingsModal (Seminar Config - model, agents, thinking)
├── MessageDetailModal (full message view)
├── Main Content (debate board/list views)
│   ├── TopicInput (initial state)
│   ├── DebateDisplay (messages with agent cards)
│   └── UserInputArea (intervention controls)
└── GoogleAuthModal (Drive backup)
```

### State Management (React hooks)
- `config`: API keys, model settings, agent customization
- `sessions`: Saved debate history (persisted to localStorage)
- `transcript`: Current debate message stream
- `agents`: Active AI personas (colors, names, styles)
- `stage`: Debate phase (idle/recruiting/debating/concluding/finished)

### Core Data Flows

**Debate Lifecycle:**
1. `startNewSession(topic)` → Moderator recruits 3-5 agents
2. `runDebateRound()` → Each agent gets context window (last 25 messages)
3. `streamLLM()` → Fetches via SSE, updates in real-time
4. `concludeDebate()` → Moderator summarizes consensus

**Context Management:**
- `contextRef` maintains conversation history
- Auto-summarization when context exceeds limits
- User interventions inserted into context stream

**Persistence:**
- localStorage for immediate state saves
- Google Drive API for cloud backups (optional)
- JSON format for full backup, Markdown for session export

### API Integration

Uses OpenAI-compatible `/chat/completions` endpoint:
- Base URL configurable (default: OpenAI)
- Streaming responses with Server-Sent Events
- AbortController for stopping debates
- Model discovery via `/models` endpoint
- Supports `reasoning_content` for deep-thinking models

## Code Conventions

**Message Roles:**
- `user`: Human intervention
- `agent`: AI council members
- `moderator`: Facilitator and summarizer
- `system`: Status updates

**Agent System:**
- Each agent gets unique color from predefined palette
- Personas generated dynamically by moderator or fallback pool
- Names/styles use Chinese characters with semantic meaning

**Error Handling:**
- Retry logic with exponential backoff (max 3 attempts)
- Graceful fallback to predefined agent pool on recruitment failure
- Abort error detection for user stops

**UI Patterns:**
- Tailwind CSS for all styling (no custom CSS)
- Modal structure with overlay/backdrop
- `memo()` for MessageCard performance
- Indigo/purple branding colors

## Localization

Primary language is **Chinese (Simplified)**. All UI text, prompts, error messages, and default agent personas are in Chinese.

## Working with This Codebase

When making changes:
1. All edits go to `index-V8.HTML`
2. Test by refreshing browser (no build step)
3. localStorage persistence - may need to clear for testing
4. Google Drive integration requires OAuth client ID in config
