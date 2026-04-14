# Human Agent

> Your personal AI coding agent — learns your taste, self-corrects, evolves.

**The agent that gets you.** Unlike generic coding assistants, Human Agent builds a model of *who you are* — your preferences, your style, your patterns — and uses that to deliver results that feel like *you* wrote them.

Unlike Hermes Agent (general-purpose, multi-platform, skill-ecosystem) or Claude Code (CLI-only, no memory), Human Agent is built around one idea: **the agent should think and work like you**.

---

## Features

| Feature | Description |
|---------|-------------|
| **Taste Learning** | Records your feedback, extracts patterns, builds a personal preference profile. The agent gets better the more you use it. |
| **Self-Correction** | When code fails, it doesn't just stop — it analyzes the error, extracts the lesson, retries with a fix. |
| **Sandbox Execution** | Runs untrusted code in Docker containers with resource limits. Falls back to local execution. |
| **Threat Detection** | Pre-execution safety scan blocks dangerous patterns (fork bombs, root deletion, SQL injection, etc.) |
| **Memory Across Sessions** | Taste profile persists in `~/.myagent/taste/` — no start-from-scratch every session. |
| **Lightweight** | Single CLI, no gateway server, no messaging platforms. Just you and the agent. |

---

## Quick Start

```bash
# Clone the repo
git clone https://github.com/YoungYang963/human-agent.git
cd human-agent

# Install dependencies
bun install

# Configure your LLM API (MiniMax via aicodee relay)
cp .env.example .env
# Edit .env with your API keys

# Initialize identity
bun run src/index.ts init

# Run your first task
./run.sh "say hi"
```

---

## Usage

```bash
# Basic task
./run.sh "fix the login bug"

# Specify language
./run.sh "write a python script to sort a list" -l python

# With test verification
./run.sh "implement quicksort" -l python -t "pytest"

# Check taste profile
./run.sh taste

# View feedback history
./run.sh feedback
```

---

## Architecture

```
human-agent
├── src/
│   ├── index.ts          # CLI entry point
│   ├── core/
│   │   ├── agent.ts     # Core agent loop (cortex + cerebellum)
│   │   └── executor.ts  # Task executor with self-correct
│   └── lib/
│       ├── anthropic.ts  # LLM client (aicodee-relay)
│       ├── tools.ts     # Tool registry (Read/Write/Edit/Glob/Grep)
│       ├── sandbox.ts   # Docker + local fallback execution
│       ├── self-correct.ts  # Self-correction engine (amygdala)
│       ├── taste.ts     # Taste profile learning
│       └── types.ts     # TypeScript types
├── run.sh               # CLI wrapper
└── docs/
    └── ARCHITECTURE.md  # Deep dive
```

**Agent Loop Flow:**
```
1. Build context (system + taste + messages)
2. Call LLM
3. If tool call → execute in sandbox → continue loop
4. If text → return to user
5. After task → update taste based on result
```

**Self-Correction Flow:**
```
1. Verify task result (run test or sandbox)
2. If failed → analyze error → extract lesson
3. Update taste profile
4. Retry with correction
5. Repeat until success or max attempts
```

---

## The Taste System

Human Agent maintains a taste profile in `~/.myagent/taste/profile.json`:

```json
{
  "userId": "default",
  "entries": [
    {
      "category": "code_style",
      "pattern": "use early returns",
      "rule": "Prefer early returns over deeply nested conditionals",
      "example": "if (!user) return error; // instead of if (user) { ... }",
      "usageCount": 12
    }
  ],
  "totalCorrections": 47
}
```

Categories: `code_style`, `naming`, `architecture`, `workflow`, `verbosity`, `general`

The agent retrieves relevant taste entries per task using keyword scoring, then injects them into the system prompt so the LLM writes *in your style*.

---

## Security

**Threat Detection** (pre-execution, `src/lib/self-correct.ts`):
- Blocks: `rm -rf /`, fork bombs, `chmod 777`, `curl|wget | sh`, SQL DROP, unconditional deletes, `eval`, `exec`

**Sandbox** (`src/lib/sandbox.ts`):
- Docker with: `--network none`, `--memory=256m`, `--memory-swap=256m`, `--pids-limit=50`
- Falls back to local execution with `timeout` command
- Each run gets ephemeral container + temp directory (cleaned up after)

---

## Configuration

**Default config** (`src/core/agent.ts`):
```typescript
{
  model: 'MiniMax-M2.7-highspeed',  // Use any OpenAI-compatible model
  maxTokens: 8000,
  temperature: 0.3,
  maxAttempts: 3,
  verificationTimeout: 30000,
}
```

**LLM Provider**: Uses `aicodee-relay` via `~/.hermes/auth.json` credential pool. Configure in `~/.hermes/.env`:

```bash
ANTHROPIC_BASE_URL=https://v2.aicodee.com
ANTHROPIC_TOKEN=your_token_here
OPENAI_API_KEY=your_key_here
```

---

## Inspired By

- [Claude Code](https://github.com/anthropics/claude-code) — streaming tool executor
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — closed-loop memory, taste architecture
- [Karpathy LLM Wiki](https://github.com/karpathy/llm-wiki) — persistent knowledge
- [OpenKB](https://github.com/VectifyAI/OpenKB) — document compilation pipeline

---

## License

MIT — do whatever you want with it. But please credit if you learned from it.

Built with curiosity by [Young Yang](https://github.com/YoungYang963).
