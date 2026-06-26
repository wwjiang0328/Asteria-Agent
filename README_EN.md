# HermesX

**A local-first AI workspace hub that lets models go beyond answering, and actually get the work done.**

HermesX understands your goals, gathers scattered context, and autonomously breaks down tasks and orchestrates tools — reading and writing files, running commands, searching the web, operating the browser and your desktop, seeing the screen and producing images and presentations — surfacing every step of progress in plain sight so ideas, information, and results flow together in one space.

Just describe the outcome you want. HermesX brings order to your conversations, files, and tasks, converging complex work into a process that is trackable, controllable, and ready to deliver. You set the direction; it drives execution and lands the result. What you get back is not a single answer, but a complete deliverable you can open, locate, and review. It also remembers your preferences and long-standing conventions, getting to know you better over time so you don't start from scratch each session. Data and execution stay local, so safety and control remain firmly on your side.

> Download: [hermesx.jackcloud.online](https://hermesx.jackcloud.online)

---

## Changelog

### v1.0.8 (2026-06-26)

- Zhiyou first-person replies: streaming responses flow directly into chat bubbles, like the user themself — no more "thinking how to reply"
- 12 Zodiac avatar system: 12 zodiac-themed avatars, no-jitter avatar bar, show only Zhiyou names (hide tool names)
- Zhiyou capability parity with direct chat: visible sub-agents, early-stop false-negative fix
- Search engine overhaul: Chinese Bing + DuckDuckGo fallback, Bing instant answer cards, noise filtering
- Browser observation: "visible key data" channel, JS widget data preserved from readability drops
- show_image tool + screenshot fixes: app-local storage for always-readable files, pre-capture window activation hint
- Anti-fake-completion: narrowed resume triggers + objective-gate checks + routing fix for self-identity queries
- Streamlined approval (3-option) + colloquial chat tone + System32 PATH fix + 4 new presentation themes

### v1.0.7 (2026-06-25)

- Fixed three Mac real-device bugs: path backslash normalization, DeepSeek reasoning_effort param, presentation deck title rendering

### v1.0.6 (2026-06-25)

- PPT slide visual overhaul: universal icon library, 3D card/blueprint layouts, hub-centered composition, background glows, 3D icon badges
- Bar chart multi-series grouping + legend, automatic data tolerance (multiple data formats)
- Presentation QA self-healing + narrower guards: auto-corrects weak model iterations instead of spinning in loops
- Humanized error messages: balance/empty-response errors shown in plain language, no raw JSON leaks
- Long-session scroll fix: auto-scroll to bottom even during thinking, new messages no longer hidden
- Academic icon expansion (flask/literature/dataset etc.), page limit raised to 30

### v1.0.5 (2026-06-24)

- New `remember` tool: the agent finally remembers your preferences — how to address you, your habits, long-standing conventions — carried across sessions, so you never repeat yourself
- More honest browser input: typing into a link or button no longer reports false success; it points to the actually-editable field and self-corrects
- No internal mechanics in chat: replies talk results and progress only — no element indices or selectors leaking through
- About page capability text upgrade: core capabilities reordered by what it can really do, clearer on the agent's true boundaries

### v1.0.4 (2026-06-24)

- Browser control breakthrough: numbered interactive elements + action-as-observation (inspired by browser-use), click returns fresh screenshot
- Browser robustness fixes: blank page fallback, slow render retry, early-stop interception, click→read false-kill fix
- Conversation title AI summarization: new conversations auto-titled by AI
- Multi-model image generation: DashScope Qwen image support + expanded adapters
- Image gallery redesign: single-image full display + multi-image grid gallery
- Patent disclosure drafter skill: from sketch to patent disclosure, agent generates it in one go
- Cross-OS command awareness + cross-session learning fixes + engine stability

### v1.0.3 (2026-06-23)

- Image generation: Agent can now generate images and embed them in conversations, with multi-model adapter support
- Markdown rendering upgrade: inline image display and richer formatting
- CommandBar interaction improvements
- HTTP proxy & Provider enhancements

### v1.0.2 (2026-06-23)

- PPT generation: Agent can now directly produce presentations (HTML Deck + PPTX), from brief to final output in one go
- Plan proposals: complex tasks now produce a plan for review before execution
- Provider rate limiting: Rate Limit Gate prevents API call overload
- RAG engine upgrade: more precise keyword retrieval
- Tool Executor refactor: Tool Registry + Safety Guards for more robust execution

### v1.0.1 (2026-06-22)

From "read-only analysis" to "hands-on action" — this release gives the agent the ability to see and operate your desktop:

- Screen vision & desktop control: capture screenshots, use mouse and keyboard to operate any desktop app or browser — open pages, click buttons, fill forms, launch software, working on-site just like a human
- Parallel sub-agent collaboration: multiple sub-agents working in parallel around the same context, streaming progress back in real time
- Windows accessibility hardening: more stable UIA element location and window traversal
- Deeper browser capture: always-on Console / Network event monitoring for more precise web interaction
- Permission panel redesign: clearer, more controllable operation permissions

Letting the agent not just "talk", but truly "do".

### v1.0.0 · Public Beta (2026-06-21)

Polished all the way from 0.x to 1.0.0, HermesX now enters public beta. This release is all about making the agent more autonomous and secure:

- Sub-agent orchestration: automatically decomposes complex tasks and spawns parallel sub-agents
- Browser tab control: locates elements via CSS/Role/Text, auto-waits for render, screenshot-to-analysis
- Persistent shell sessions: directory and environment carry across commands
- Project context awareness: reads HERMESX.md to inject instructions on the fly
- Disaster protection: critical deletes are undoable — no more panic
- Install integrity checks + network safety hardening
- MCP ecosystem: GitHub / PostgreSQL and more plug-and-play tools

Give it a try and tell us what you think. Every piece of feedback makes the full release better.
