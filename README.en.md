# HermesX

<div align="center">

**Desktop Agent Workbench**

[![Tauri](https://img.shields.io/badge/tauri-2-ffc131)](src-tauri/Cargo.toml)
[![React](https://img.shields.io/badge/react-19-61dafb)](package.json)
[![TypeScript](https://img.shields.io/badge/typescript-6.0-3178c6)](package.json)
[![Rust](https://img.shields.io/badge/rust-1.95+-orange)](src-tauri/Cargo.toml)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

[中文](./README.md)

</div>

---

## Project Positioning

HermesX is a Tauri-based desktop AI workbench that connects Hermes backend capabilities, local AI model configurations, local file tools, a task execution panel, a memory system, MCP tools, and desktop pet feedback.

Its core goal is not to build yet another chat box, but to put "conversations, tasks, tools, files, permissions, and Agent progress" into a single native workbench. Remote Agents can participate in analysis and execution, but local file access, writes, command execution, and permission confirmation are always controlled by HermesX.

The project is in an early desktop application stage — features are functional but iterating rapidly. This README reflects the actual capabilities of the current codebase. Roadmap items and experimental ideas are not presented as shipped features.

## Main Features

### Conversation Workbench

- Direct AI model Q&A with configured providers.
- `@agent` to dispatch a task to a specific Agent, or `@all` to broadcast to all online Agents.
- Task acceptance status is shown immediately upon dispatch, preventing blank waiting.
- Attachments (images, files) are supported and saved to HermesX's local data directory.
- Local file, directory, and command requests go through HermesX's native context and permission flow.

### Agent Collaboration

- Connect multiple Hermes instances and aggregate their Agent configurations.
- The session sidebar displays instances, online status, and Agent counts.
- Precisely target Agents by instance/name to avoid name clashes.
- Agent task progress, logs, and results flow back into the current session.
- When a remote Agent requests a local tool, HermesX executes it on the user's machine and returns the result.

### Local Tools & Permission Boundary

HermesX provides a native tool execution layer. Common capabilities include:

| Category | Capabilities |
| --- | --- |
| Files | read, write, read directory, move, create directory, delete path |
| Search | file name search, text search, path check |
| Commands | execute shell commands in the current workspace |
| Git | status, diff, log, commit, branch operations |
| Desktop | accessibility tree snapshot, window list |
| MCP | connect external MCP Servers and register tools |

Security is native-first:

- The workspace is the default context for local file operations.
- High-risk operations (write, delete, move, command execution) require confirmation.
- Sensitive paths (`.git`, `.env`, key files, system directories) are restricted or blocked.
- Remote Agents cannot use their own server directory as HermesX's local workspace.

### Code Indexing & Memory System

The memory system is a multi-layered data management architecture, not a single chat log table:

- **Code Indexing / RAG**: Scan the selected workspace, chunk and embed files for semantic search and Agent retrieval.
- **White-box Memory**: Memory entries include type, tags, strength, version, TTL, and source task fields. All memories are viewable, editable, and filterable in the settings panel.
- **Dream Engine**: Automatically and periodically cleans up expired memories, merges highly similar entries, and archives low-strength stale data. Dream results are visible and support one-click rollback to the previous version.
- **Lessons Learned**: Extract usage patterns, failure information, and conversation summaries from tool calls and task results.
- **Project Knowledge**: Record project conventions, tech stack details, and common errors.
- **Agent Action Diary**: Record agent tool call traces for debugging and pattern analysis.

In Settings → Memory System you can:

- View all memory entries (filter by type/tags, edit and delete directly).
- Select a workspace and build a code index.
- Manage Dream maintenance parameters (interval, trigger thresholds, etc.).
- View Dream history and roll back.
- View retrieval traces and agent action diary.

Index data is written to HermesX's local SQLite database, not to the project directory.

Default database location (Tauri app local data directory):

```text
app_local_data_dir()/memory.db
```

On Windows:

```text
C:\Users\<username>\AppData\Local\com.hermes.hermesx\memory.db
```

SQLite WAL mode is enabled, so `memory.db-wal` and `memory.db-shm` may appear in the same directory (this is normal).

### Workflows

HermesX includes a visual workflow panel:

- Create workflows from templates.
- View and adjust nodes on a canvas.
- Configure models, Agents, inputs, prompts, output variables, and output paths per node.
- Execution progress is shown in a right-side drawer without polluting normal conversations.
- Node states: `pending`, `running`, `completed`, `failed`, `skipped`.

### MCP Tools

The Settings center includes MCP management for adding, connecting, and stopping MCP Servers. Connected MCP tools enter HermesX's dynamic tool registry for use by local execution and Agent tasks.

Built-in presets include:

- Filesystem
- GitHub
- Fetch
- Memory
- PostgreSQL
- GSAP

### Long-Running Tasks & Run Management

HermesX handles Hermes Agent task execution with SSE + polling dual-mode:

- **SSE Streaming Execution**: Tasks submitted via `/v1/runs` default to SSE event stream for real-time progress.
- **Auto-Polling on Disconnect**: When SSE unexpectedly disconnects (network flapping, Agent restart), HermesX automatically switches to polling mode, querying task status every 30 seconds for up to 2 hours.
- **exitReason Tracking**: Complete recording of task termination reasons (sse_timeout → polling_completed / polling_failed / polling_cancelled), enabling precise debug of disconnection points.
- **Managed Runs Persistence**: In-flight run records are persisted; status is recoverable after app restart.
- **Run Sweep**: Periodically cleans up expired and completed run records to prevent data accumulation.

This mechanism transforms HermesX from "short tasks only" to "waiting tens of minutes for long tasks" — users don't need to stare at the screen waiting for results.

### Desktop Pet

The desktop pet provides task status feedback:

- Independent transparent always-on-top window.
- Character selection, size, position, and behavior settings.
- States: idle, thinking, working, success, failure.
- Real animation frames are previewable in the settings page.

## Technical Architecture

HermesX consists of a React frontend and Tauri/Rust backend.

```text
src/
  components/                 React UI
    Workspace.tsx             Conversation workbench
    CommandBar.tsx            Input bar, model selection, @agent dispatch
    Sidebar.tsx               Collaboration instance sidebar
    Settings.tsx              Settings center
    WorkflowBuilder.tsx       Workflow canvas
    settings/MemoryPanel.tsx  Memory system & code indexing

  services/                   Frontend service layer
    providers.ts              AI Provider invocation
    toolRegistry.ts           Tool definitions
    toolExecutor.ts           Tool execution & permission handling
    localToolAgent.ts         Local tool Agent loop
    ragEngine.ts              Code indexing & semantic search
    crossSessionLearning.ts   Cross-session learning
    workflowEngine.ts         Workflow execution
    context.ts                Context management & memory versioning
    addOnlyMemory.ts          White-box memory write & retrieval
    memoryMaintenance.ts      Dream engine (cleanup/merge)
    managedRuns.ts            Hermes Run lifecycle & polling
    mcp/                      MCP client & manager

  store/                      Zustand state
    useHermesXStore.ts        Global config, workspace, memory
    useConversationStore.ts   Conversations & messages
    useAgentConnectionStore.ts Instance connection status
    useAgentExecutionStore.ts Agent task execution

src-tauri/
  src/
    main.rs                   Tauri startup & command registration
    commands/                 Tauri commands
      fs.rs                   Local filesystem commands
      http.rs                 HTTP/SSE proxy
      memory.rs               Memory database commands
      data.rs                 Local data directory & attachments
      watch.rs                File watching
      pet.rs                  Desktop pet commands
    memory.rs                 SQLite memory store
    data_dir.rs               App data directory management
    mcp_server.rs             HermesX local MCP service
    watch_manager.rs          File watch management
```

## Technical Background & Design Basis

HermesX's design does not invent concepts from scratch. It consolidates mature practices from desktop Agents, coding Agents, long-term memory, skill discipline, GUI perception, and local security boundaries into a single native desktop workbench. The projects listed below are not dependency lists or endorsements — they explain which problems HermesX paid attention to during design and how the current codebase addresses them.

### Desktop Agents & GUI Perception

| Project | Focus | Corresponding Issue in HermesX |
| --- | --- | --- |
| agent-desktop | Accessibility tree traversal, structured UI tree, on-demand deep reading | Desktop perception should not rely solely on screenshots; HermesX exposes structured desktop context via `a11y_snapshot`, `a11y_list_windows`. |
| Cua | Background desktop control, avoid disrupting user focus | HermesX's desktop perception focuses on reading and context provision; it does not take over mouse, focus, or user operations by default. |
| Microsoft UFO | Multi-Agent division of labor, Host/App hierarchy, Windows UI Automation | HermesX's task scheduling, workflows, and Agent dispatch are organized around "decomposition, dispatch, and result return." |
| OpenAdapt | Desktop event recording and replay | HermesX's file watching, task logs, and execution panel focus on observability and traceability. |
| OS-Copilot | OS-level Agent, self-improvement loop | HermesX's lessons learned, project knowledge, and task history provide better native context for subsequent tasks. |
| AgentDesk | Isolated environments and remote desktop-style Agent workspaces | HermesX does not treat remote environments as local projects; it keeps local file permissions on the desktop. |
| OpenPawz | Tauri desktop Agent, Fleet-style multi-Agent orchestration | HermesX uses Tauri/Rust desktop architecture and aggregates multi-instance, multi-Agent into the session workbench. |

### Coding Agents & Tool Loops

| Project | Focus | Corresponding Issue in HermesX |
| --- | --- | --- |
| Claude Code | Tool call loop, context compression, subtask delegation | HermesX's local tool Agent, remote tool relay, and context compression revolve around "model reasoning, local execution." |
| Cline | In-IDE file editing, command execution, user confirmation | HermesX's file read/write, command execution, and authorization cards follow the principle "tool actions must be visible." |
| Aider | Repository structure understanding, multi-file editing, Git assistance | HermesX's project context, Git tools, and code indexing target real project workspaces. |
| Goose | Rust Agent framework, MCP/ACP extensions | HermesX's backend uses Rust/Tauri and connects external tools via the MCP management page. |
| OpenHands | Agent runtime environment, skills, and task workspace | HermesX's execution panel, workflow canvas, and tool relay provide a desktop-side task execution surface. |
| Continue | Model switching, context providers, IDE assistance | HermesX's Provider management, model selector, and native context gathering target multi-model usage scenarios. |

### Memory, Context & Skill Discipline

| Project | Focus | Corresponding Issue in HermesX |
| --- | --- | --- |
| Mem0 | ADD-only memory, entity and time signals, long-term memory retrieval | HermesX preserves append-only memory, lessons learned, and project knowledge without collapsing historical experience into a single latest summary. |
| agentmemory | Typed memory, version chains, context priority | HermesX's memory entries include type, tags, strength, version, TTL, and source task fields. |
| Letta | Agent state persistence, memory block management | HermesX separates user preferences, project facts, lessons learned, and code indexing into distinct stores to avoid mixing all context in chat history. |
| screenpipe | Local activity awareness, long-term recording, privacy boundaries | HermesX's logs, file watching, and desktop awareness prioritize local data directories and the native permission model. |
| OmniParser | GUI screenshot to structured elements | HermesX's primary path is the accessibility tree; visual parsing can be a future supplement rather than a replacement for native structured context. |
| WindowsAgentArena | Desktop Agent evaluation dimensions | HermesX's execution panel and log structure preserve data foundations for future task quality assessment. |
| obra/superpowers | Agent workflow discipline, TDD, planning, review | HermesX's skill system supports on-demand loading of process skills to help Agents maintain execution discipline in complex tasks. |
| everything-claude-code | Self-debugging, error patterns, verification loop | HermesX includes built-in `verifier`, `selfHealing`, error patterns, and related check capabilities. |

## Key Implementation Trade-offs

This section explains how the above design basis lands in HermesX's current code. It is not a roadmap — it describes engineering structures that already exist or are actively used in the current implementation.

### Tauri 2 & Rust Backend

HermesX's early form was closer to a Web/Electron-style desktop application, but the mainline has moved to Tauri 2. This is not about pursuing a "cooler tech stack" — it's about placing local files, processes, SQLite, PTY, system tray, desktop pet windows, and HTTP/SSE proxies within clearer native boundaries.

Current backend modules:

- `commands/fs.rs`: local filesystem read/write, search, path checking.
- `commands/http.rs`: model request and SSE request proxy, reducing browser environment constraints.
- `commands/memory.rs` & `memory.rs`: SQLite memory database, full-text search, tool statistics.
- `commands/watch.rs` & `watch_manager.rs`: file watching.
- `commands/data.rs` & `data_dir.rs`: app local data directory, attachments, and document storage.
- `mcp_server.rs`: HermesX local MCP service.
- `pet.rs`: desktop pet window and status feedback.

The frontend exposes a unified bridge interface via `runtime.ts`. React components do not directly reference Tauri command names; they use the bridge API under `window.hermesX`.

### Remote Agent & Local Execution Separation

HermesX treats remote Hermes Agents as capability backends, not project execution environments. This boundary is critical:

- Remote Agents can analyze, plan, and generate.
- Remote Agents cannot use their own server directory as the user's project.
- Reading, searching, writing, and command execution must go through HermesX's local tool layer.
- HermesX decides whether to execute based on workspace, authorization records, and sensitive path rules.

Corresponding code paths:

- `useAgentConnectionStore.ts`: instance connection and Agent status.
- `useAgentExecutionStore.ts`: Agent task creation, execution, failure, and completion return.
- `remoteAgentToolBridge.ts`: bridging remote Agent requests for local tools.
- `agentBriefing.ts`: writes native boundaries, session context, memory, and tool rules into the task briefing before dispatch.

### Local Tool Relay

HermesX's tool system is not a set of scattered helper buttons — it is uniformly registered, executed, budgeted, and displayed.

- `toolRegistry.ts` describes tool capabilities, parameters, and permission levels.
- `toolExecutor.ts` handles parameter validation, permission checks, execution, and result enrichment.
- `toolBudget.ts` limits tool result length to prevent long files or command outputs from overwhelming context.
- `localToolAgent.ts` allows local models to process native tasks via a "think → tool call → observe → continue" loop.

This structure allows ordinary AI conversations, remote Agent tool requests, workflow nodes, and MCP tools to share the same execution boundary.

### Memory System & Code Indexing

HermesX's memory is a combination of multiple data stores, not a single field:

- Persisted state (`persistedState.ts`): app config, memory maintenance status, Dream history.
- Context versioning (`context.ts`): memory entries carry version numbers; on write, Jaccard similarity automatically marks older entries as superseded. Supports rollback to any Dream history snapshot.
- Dream engine (`memoryMaintenance.ts`): runs periodically, supporting three strategies: expiration cleanup, similarity merge, and low-strength archival. Configurable trigger interval, minimum write threshold, and cooldown time.
- SQLite `memory.db`: code index, lessons learned, conversation summaries, project knowledge, tool statistics.
- In-memory cache: runtime index, agent action diary, task result cache.

RAG code indexing is performed in `ragEngine.ts`:

1. Scan the user-selected workspace.
2. Ignore `node_modules`, `.git`, `dist`, `build`, `target`, etc.
3. Chunk supported text/code files.
4. Generate embeddings for each chunk.
5. Write to SQLite via Tauri `memory_store` with type `code_chunk`.

Cross-session learning in `crossSessionLearning.ts` includes:

- Extracting success patterns and failure lessons from tool calls.
- Saving conversation summaries.
- Tracking tool usage success rates.
- Retrieving relevant historical experience in subsequent tasks.

### Context Compression & Multi-Agent Routing

The most common problem in long tasks and multi-Agent collaboration is that logs grow increasingly long and the model sees increasingly cluttered context. HermesX addresses this with layered processing:

- Tool result summarization: long file reads, command outputs, and tool results are structurally trimmed.
- Session context compression: history messages are compressed against budget, preventing irrelevant content from occupying the window.
- Multi-Agent context routing: one Agent does not directly read all other Agents' full logs — it sees summaries relevant to the current task.

Corresponding modules:

- `contextCompressor.ts`
- `multiAgentContext.ts`
- `conversationTimeline.ts`
- `agentBriefing.ts`

### Long-Running Run Management & Polling Fallback

HermesX doesn't just "submit a task and wait on a dead SSE stream" — it provides full lifecycle encapsulation of the Hermes Agent `/v1/runs` API:

- **SSE + Polling Dual-Mode**: Default SSE streaming for real-time events; on unexpected disconnect (network flapping, Agent restart), automatically switches to 30-second-interval polling via `GET /v1/runs/:id`.
- **Phased exitReason**: Distinguishes SSE-phase reasons (`sse_timeout`, `sse_disconnected`) from polling-phase reasons (`polling_completed`, `polling_failed`, `polling_cancelled`). The former is never overwritten by the latter, enabling accurate diagnosis of the actual disconnection cause.
- **2-Hour Hard Cap**: Polling lasts at most 2 hours (`HERMES_RUN_POLL_MAX_DURATION_MS`), preventing indefinite client polling when the Agent service-side task never terminates.
- **Managed Runs Persistence**: All run records (running, completed, failed) are persisted, with taskId fallback deduplication and periodic sweep of expired records.

Corresponding modules:

- `src/api/adapters/hermes.ts`: SSE stream, polling loop, exitReason tracking.
- `src/services/managedRuns.ts`: Run record persistence, deduplication, and sweep.
- `src/constants.ts`: Timeout and polling interval constants.

### Workflow Execution Isolation

Workflows are not another display format for ordinary chat — they are an independent task execution surface:

- `WorkflowBuilder.tsx` provides templates, canvas, node configuration, and a run drawer.
- `workflowEngine.ts` executes the DAG by node dependencies.
- `useWorkflowStore.ts` preserves workflow state.
- `useAgentExecutionStore.ts` supports `silent` tasks, keeping workflow results in the workflow drawer rather than polluting the conversation area.

This keeps ordinary conversations clean while workflow tasks remain observable and reviewable.

### Skill System

HermesX's skills are not lengthy text manually copied into prompts — they enter the Agent workflow through a skill directory and on-demand loading mechanism.

Current skill sources include:

- Project-level skill directory.
- User-level skill directory.
- Skills provided by plugins or runtime.

Key modules:

- `skillsEngine.ts`: loads and builds the skill directory.
- `skillRouting.ts`: matches tasks to skills.
- `skillCurator.ts`: prompts for knowledge consolidation after task completion.
- `toolRegistry.ts` / `toolExecutor.ts`: provide tool entry points such as `hermesx_load_skill`.

The goal of skills is not to make the Agent "appear smarter" — it is to help it follow operational discipline more reliably across design, implementation, debugging, verification, and review flows.

## Data Storage

HermesX stores runtime data in the Tauri app local data directory. The actual directory path can be viewed in the relevant application page.

| Data | Storage Location |
| --- | --- |
| App config | Tauri Store / HermesX persisted config |
| Conversation history | HermesX native persistent storage |
| Attachments, images, generated docs | `storage/images`, `storage/documents` under app data dir |
| Code index, lessons, project knowledge | `memory.db` SQLite database |
| Tool call logs | In-app log system & SQLite statistics |

Code index data is stored in the `memories` table with type `code_chunk`. Embeddings are written to the database as BLOBs and are not written to the indexed project directory.

## Quick Start

### Prerequisites

- Node.js 20+
- npm
- Rust 1.95+
- System WebView runtime

Windows typically requires Visual Studio Build Tools and Windows SDK. Linux requires WebKitGTK, GTK, AppIndicator, and other Tauri dependencies.

### Install Dependencies

```bash
npm install
```

### Start Desktop Dev Environment

```bash
npm run dev
```

Equivalent:

```bash
npm run tauri:dev
```

### Start Web Frontend Only

```bash
npm run dev:web
```

This mode is suitable for debugging the frontend UI. Use the desktop dev environment for features involving Tauri IPC, local files, desktop pet window, etc.

## Build

### Build Desktop App

```bash
npm run build
```

Or:

```bash
npm run tauri:build
```

### Build Frontend Assets

```bash
npm run build:web
```

Build output locations are determined by Tauri. Common paths include:

```text
src-tauri/target/release/
src-tauri/target/release/bundle/
```

The Windows NSIS installer is typically located at:

```text
src-tauri/target/release/bundle/nsis/
```

## Tests & Checks

HermesX includes frontend and backend tests:

```bash
npm run test          # Frontend TypeScript tests (Vitest)
npm run build:web     # Frontend build check
```

Frontend test coverage includes: tool registry, tool executor, path approval, memory writes, memory maintenance, and context management.

Rust check:

```bash
cd src-tauri
cargo check
```

## Configuration

### AI Provider

Configure Provider in Settings (OpenAI-compatible, Anthropic, Ollama, or custom endpoints). The session workbench can select the currently active model.

### Hermes Instances

Add Hermes instance URLs in Connection settings. Once connected, instances' Agents appear in the collaboration sidebar for `@agent` dispatch.

### Auto-Update

On startup, HermesX automatically connects to the `hermesX-admin` version management platform to check for new releases. The Settings center displays the current version and changelog, with one-click access to download the latest installer.

### Workspace

The workspace is HermesX's primary context for local projects. It affects:

- Default directory for file reads and searches.
- Default working directory for command execution.
- Target directory for code indexing.
- Native project boundary in Agent briefings.

The workspace can be set via the directory button in the conversation input bar, or selected in Settings → Memory → Code Indexing.

### MCP

Add server commands, parameters, and environment variables in the MCP settings page. Tools are dynamically registered upon successful connection. MCP Servers that require access to external services may need additional environment variables or credentials.

## FAQ

### Why does code indexing show 0?

Common reasons:

- No workspace selected yet.
- The selected directory is not a project root.
- No supported text or code files in the directory.
- Files are in ignored directories such as `node_modules`, `.git`, `dist`, `build`, `target`.
- Embedding or database write failed.

Commonly supported extensions:

```text
.ts .tsx .js .jsx .vue .svelte .rs .py .go .java .c .cpp .h .hpp
.css .scss .less .html .json .yaml .yml .md .txt .sh .bash .sql
```

### Why are there no new files in my project after indexing?

This is normal. Index data is written to HermesX's `memory.db`, not to the indexed project directory.

### What's the difference between `@agent` and direct AI Q&A?

Direct AI Q&A uses the currently selected model for replies. `@agent` dispatches the task to a connected Hermes Agent, attaching the current session context, workspace, file context, and permission boundaries. When the Agent needs local files or commands, it requests local tool execution through HermesX.

### When is authorization required?

Operations such as reading unauthorized paths, writing files, deleting files, moving paths, and executing commands may require confirmation. Authorization is controlled by HermesX, not decided by the remote Agent.

### Can I just use it as a regular AI chat tool?

Yes. Configure an AI Provider and start asking questions directly in the session workbench. Hermes instance connections are not required for basic model Q&A.

## Current Limitations

- Some desktop awareness features depend on OS support and platform adaptation.
- Stable execution of remote Agents depends on the connection status and interface capabilities of the corresponding Hermes instance.
- Code indexing currently uses local SQLite storage, suitable for project-level retrieval but not a general-purpose large-scale vector database.
- In polling mode, tasks wait at most 2 hours; manual re-submission is needed after timeout.
- Workflow, memory, and MCP modules are still being iterated; configuration formats and interfaces may continue to change.

## Contributing

Issues, suggestions, and pull requests are welcome. Before submitting code, it is recommended to run at least:

```bash
npm run build:web
```

For Rust backend changes, also run:

```bash
cd src-tauri
cargo check
```

## License

MIT © [wwjiang0328](https://github.com/wwjiang0328)
