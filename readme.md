# OpenClaw Token Optimization Techniques - Complete Analysis

> **Author**: User A · Agent-X  
> **Last Updated**: 2026-05-19  
> **Applicable Version**: OpenClaw 2026.4.23+  
> **Target Audience**: Technical personnel interested in AI Agent / LLM cost optimization  

---

## Table of Contents

- [OpenClaw Token Optimization Techniques - Complete Analysis](#openclaw-token-optimization-techniques---complete-analysis)
  - [Table of Contents](#table-of-contents)
  - [Part 1: Principles - Hidden Costs of System Prompts](#part-1-principles---hidden-costs-of-system-prompts)
    - [Bootstrap File Loading Mechanism](#bootstrap-file-loading-mechanism)
    - [Context Window and Compaction Mechanism](#context-window-and-compaction-mechanism)
    - [Fixed Overhead per New Session](#fixed-overhead-per-new-session)
  - [Part 2: Testing - Bootstrap File Quantitative Analysis](#part-2-testing---bootstrap-file-quantitative-analysis)
    - [File Volume Before Optimization](#file-volume-before-optimization)
    - [File Volume After Optimization](#file-volume-after-optimization)
    - [Cumulative Consumption Comparison by Usage Scenario](#cumulative-consumption-comparison-by-usage-scenario)
  - [Part 3: Optimization - Seven Core Techniques](#part-3-optimization---seven-core-techniques)
    - [1. Tree-Structured Document Architecture (Old: Single File → New: Multi-Layer Index)](#1-tree-structured-document-architecture-old-single-file--new-multi-layer-index)
      - [Optimization Principle](#optimization-principle)
      - [Measured Data](#measured-data)
      - [Cost Savings (Monthly)](#cost-savings-monthly)
      - [Resource Consumption Changes](#resource-consumption-changes)
    - [2. AI Auto-Compression (Compaction)](#2-ai-auto-compression-compaction)
      - [Optimization Principle](#optimization-principle-1)
      - [Measured Comparison](#measured-comparison)
      - [Cost Savings](#cost-savings)
      - [Resource Consumption Changes](#resource-consumption-changes-1)
    - [3. Local Model Management of Lightweight Tasks (QMD / Ollama)](#3-local-model-management-of-lightweight-tasks-qmd--ollama)
      - [Optimization Principle](#optimization-principle-2)
      - [QMD Application](#qmd-application)
      - [Measured Data](#measured-data-1)
      - [Cost Savings](#cost-savings-1)
      - [Resource Consumption Changes](#resource-consumption-changes-2)
    - [4. Direct Script-to-API Calls, Bypassing Bootstrap](#4-direct-script-to-api-calls-bypassing-bootstrap)
      - [Optimization Principle](#optimization-principle-3)
      - [Measured Data](#measured-data-2)
      - [Resource Consumption Changes](#resource-consumption-changes-3)
    - [5. Console Commands Replace LLM Conversation](#5-console-commands-replace-llm-conversation)
      - [Optimization Principle](#optimization-principle-4)
      - [Practical Application](#practical-application)
      - [Resource Consumption Changes](#resource-consumption-changes-4)
    - [6. Daily Logic CPU-fication (Python Cron Direct Push)](#6-daily-logic-cpu-fication-python-cron-direct-push)
      - [Optimization Principle](#optimization-principle-5)
      - [Implemented CPU-fied Tasks](#implemented-cpu-fied-tasks)
      - [Measured Comparison](#measured-comparison-1)
      - [Technical Implementation](#technical-implementation)
      - [Resource Consumption Changes](#resource-consumption-changes-5)
    - [7. Intelligent Demands Pulled Back from LLM to CPU (Heartbeat Checklist-ification)](#7-intelligent-demands-pulled-back-from-llm-to-cpu-heartbeat-checklist-ification)
      - [Optimization Principle](#optimization-principle-6)
      - [Transformation Comparison](#transformation-comparison)
      - [Measured Data](#measured-data-3)
      - [Cost Savings](#cost-savings-2)
      - [Resource Consumption Changes](#resource-consumption-changes-6)
  - [Comprehensive Benefit Assessment](#comprehensive-benefit-assessment)
    - [Monthly Cost Comparison Summary](#monthly-cost-comparison-summary)
    - [Annualized Comparison](#annualized-comparison)
    - [Beyond Just Saving Money](#beyond-just-saving-money)
  - [Appendix 1: Model Pricing Reference](#appendix-1-model-pricing-reference)
  - [Appendix 2: Vectorization of Skill Descriptors](#appendix-2-vectorization-of-skill-descriptors)
  - [Conclusion](#conclusion)

---

## Part 1: Principles - Hidden Costs of System Prompts

### Bootstrap File Loading Mechanism

Each time `/new` or `/reset` is executed to create a new session, the OpenClaw runtime automatically loads the following content as **System Prompt + Startup Context**:

| File | Loading Method | Purpose |
|------|----------------|---------|
| `AGENTS.md` | System Prompt Injection | Agent behavior instruction tree |
| `SOUL.md` | System Prompt Injection | Personality definition |
| `USER.md` | System Prompt Injection | User information |
| `HEARTBEAT.md` | System Prompt Injection | Scheduled task checklist |
| `TOOLS.md` | System Prompt Injection | Local tool configuration |
| `MEMORY.md` | Startup Context | Long-term memory |
| `memory/*.md` (past 2 days) | Startup Context | Daily work logs (≤2800 characters) |

These files are **not visible in the conversation history**, but **consume actual context window**. Every LLM inference must process this content.

### Context Window and Compaction Mechanism

OpenClaw's compaction mechanism uses a `mode: safeguard` strategy:

- **Trigger Condition**: Automatically triggered when conversation history + bootstrap approach the context limit
- **Compression Method**: Generate summaries of early conversations, retain recent details
- **Problem**: If the bootstrap file itself is large, less space remains for actual conversations, compaction triggers more frequently, and each compaction consumes tokens

### Fixed Overhead per New Session

Using the default model MiniMax M2.7 (200K context window) as an example:

> **Before Optimization**: bootstrap ~25,000 bytes ≈ ~6,250 tokens  
> **After Optimization**: bootstrap ~8,300 bytes ≈ ~2,075 tokens  
> 
> Each session startup saves **~4,175 tokens**, not including subsequent chain effects of compaction in conversations.

The same principle applies to models like DeepSeek V3.2 (200K context). If your daily usage involves frequent `/new` / `/reset` (e.g., task switching, context cleanup), the savings double.

---

## Part 2: Testing - Bootstrap File Quantitative Analysis

> All data below are based on actual file measurements. Sensitive content has been anonymized: usernames → "User A", Agent names → "Agent-X".

### File Volume Before Optimization

| File | Lines | Bytes | Estimated Tokens | Main Content |
|------|-------|-------|------------------|--------------|
| AGENTS.md | ~300 | ~12,000 | ~3,000 | Behavior rules, skill index, memory rules, quick decisions all mixed |
| MEMORY.md | ~200 | ~8,000 | ~2,000 | Holdings info, built systems, technical architecture, user goals |
| SOUL.md | 36 | 1,673 | ~418 | Personality definition |
| USER.md | 11 | 278 | ~70 | Username/timezone/preferences |
| TOOLS.md | 34 | 827 | ~207 | Search toolchain, local configuration |
| HEARTBEAT.md | 28 | 1,681 | ~420 | Heartbeat checklist |
| **Total** | **~609** | **~24,459** | **~6,115** | |

### File Volume After Optimization

| File | Lines | Bytes | Estimated Tokens | Change |
|------|-------|-------|------------------|--------|
| AGENTS.md | 56 | 2,278 | ~570 | ⬇️ **-81%** |
| MEMORY.md | 62 | 1,589 | ~397 | ⬇️ **-80%** |
| SOUL.md | 36 | 1,673 | ~418 | — |
| USER.md | 11 | 278 | ~70 | — |
| TOOLS.md | 34 | 827 | ~207 | — |
| HEARTBEAT.md | 28 | 1,681 | ~420 | — |
| **Total** | **~227** | **~8,326** | **~2,082** | ⬇️ **-66%** |

> Extracted detailed rules moved to `docs/` subdirectory (5 files, 9,452 bytes total), loaded on-demand by LLM via `read` tool, no longer injected with bootstrap.

### Cumulative Consumption Comparison by Usage Scenario

Assuming typical usage patterns:
- **Daily Conversation**: 10 rounds per day, average 500 tokens input + 200 tokens output per round
- **Lightweight Tasks**: 2 tasks per day, 3,000 tokens context each
- **Session Rebuild**: ~3 times per day `/new` or `/reset`

**Monthly Consumption Before Optimization**:
```
Daily conversation: 10 × 700 = 7,000 tokens
Daily tasks: 2 × 3,000 = 6,000 tokens
Bootstrap loading (×3): 3 × 6,115 = 18,345 tokens
────────────────────────────
Daily total: 31,345 tokens
Monthly total: 31,345 × 30 ≈ 940,350 tokens ≈ 0.94M tokens
```

**Monthly Consumption After Optimization**:
```
Daily conversation: 10 × 700 = 7,000 tokens
Daily tasks: 2 × 3,000 = 6,000 tokens  
Bootstrap loading (×3): 3 × 2,082 = 6,246 tokens
────────────────────────────
Daily total: 19,246 tokens
Monthly total: 19,246 × 30 ≈ 577,380 tokens ≈ 0.58M tokens
```

**Bootstrap optimization alone saves ~0.36M tokens/month**. Combined with other optimization techniques, total savings far exceed this number.

---

## Part 3: Optimization - Seven Core Techniques

### 1. Tree-Structured Document Architecture (Old: Single File → New: Multi-Layer Index)

#### Optimization Principle

The more content crammed into an AI Agent's system prompt, the more it must "read before acting" for each inference. Traditional approaches mix all rules, indexes, and memories into one large file (e.g., AGENTS.md with 300 lines), requiring the LLM to process all 300 lines before thinking about your problem.

**Solution**: Shrink AGENTS.md and MEMORY.md to index files (<60 lines), split detailed rules by module into `docs/` subdirectory. The LLM sees only the index at startup and reads specific documents on demand.

```
workspace-qqclaw/
├── AGENTS.md          (56 lines) ← Top-level index, contains document tree
├── MEMORY.md          (62 lines) ← Summary memory
├── docs/
│   ├── OPENROUTER.md  (68 lines)
│   ├── WEB-SEARCH.md  (43 lines)
│   ├── MEMORY-SYSTEM.md (64 lines)
│   ├── TRADE-MONITOR.md (97 lines)
│   └── MULTI-SEARCH.md  (94 lines)
```

#### Measured Data

| Metric | Before Optimization | After Optimization | Savings |
|--------|---------------------|-------------------|---------|
| AGENTS.md tokens | ~3,000 | ~570 | **81%** |
| MEMORY.md tokens | ~2,000 | ~397 | **80%** |
| Bootstrap Total | ~6,115 | ~2,082 | **66%** |
| Total Documentation | ~24,459 bytes | ~17,778 bytes (incl. docs/) | Comparable lines, structural optimization |

#### Cost Savings (Monthly)

Based on Sonnet ($9/MT average) calculation:

```
Before: 6,115 tokens × 3 sessions/day × 30 days = 550,350 tokens × $9/MT = $4.95/month (bootstrap only)
After: 2,082 tokens × 3 sessions/day × 30 days = 187,380 tokens × $9/MT = $1.68/month (bootstrap only)
Savings: $0.27/month
```

Seems small? But this is just the bootstrap single-point saving. The real payoff: **smaller system prompts mean each conversation processes ~4,000 fewer tokens, compaction triggers less frequently, and more conversation rounds fit in the window.**

#### Resource Consumption Changes

| Resource | Change |
|----------|--------|
| Context per-round inference | ⬇️ ~4,000 tokens |
| Compaction trigger frequency | ⬇️ Delayed trigger (longer effective conversation) |
| LLM response latency | ⬇️ Slight decrease (reduced prompt processing) |
| Documentation maintainability | ⬆️ Improved (modular, changes don't affect others) |

---

### 2. AI Auto-Compression (Compaction)

#### Optimization Principle

OpenClaw's compaction mechanism is like "automatic conversation history summarization":

- When conversation history + system prompt approach the context limit, early conversations are compressed into summaries
- In `mode: safeguard`, the system automatically triggers at the safety boundary
- Compressed content is preserved in summary form, freeing space for new conversations

**Why does this save tokens?** If your context window is 200K tokens, without compression, each inference must process the full conversation history (possibly 50K-100K tokens). After compression, early history becomes a 1K-2K summary, and new conversations only need 10K-30K tokens.

#### Measured Comparison

| Scenario | No Compression | With Compression (safeguard) |
|----------|----------------|------------------------------|
| Context after 100 rounds | ~120,000 tokens | ~25,000 tokens |
| Per-round token consumption | ~1,200 (full) | ~600 (summary + new round) |
| LLM failure critical point | ~170 rounds | Theoretically infinite |

#### Cost Savings

Based on Haiku long conversation scenario:

```
No compression: 100 rounds × 1,200 tokens/round = 120,000 tokens/day × $9/MT × 30 days = $32.4/month
With compression: 100 rounds × 600 tokens/round = 60,000 tokens/day × $9/MT × 30 days = $16.2/month
Savings: $1.33/month
```

Calculated per day with 100 conversation rounds. Combined with other optimizations, total savings are even more impressive.

#### Resource Consumption Changes

| Resource | Change |
|----------|--------|
| GPU (inference burden) | ⬇️ Prompt processing reduced ~50% |
| Context utilization | ⬆️ Improved (more effective conversations) |
| Summary quality | ⚠️ Summaries may lose details (safeguard mode is conservative, low risk) |

---

### 3. Local Model Management of Lightweight Tasks (QMD / Ollama)

#### Optimization Principle

Not all tasks require large models. OpenClaw supports a tiered model strategy:

| Task Type | Model | Cost | Notes |
|-----------|-------|------|-------|
| Heartbeat Detection | Ollama qwen2.5:3b | **$0** | Local CPU inference |
| Security Audit | Ollama qwen2.5:3b | **$0** | Scans every 5 minutes |
| Memory Retrieval | QMD 2.1.0 | **$0** | Local semantic search |
| Complex Conversation | MiniMax / DeepSeek | Paid | Only complex tasks via cloud |

#### QMD Application

QMD (v2.1.0) is a local embedded vector retrieval engine for semantic search in the memory system:

- **Builtin**: SQLite + FTS5 full-text search (has latency)
- **QMD**: Standalone sidecar process, local vector search (zero latency, zero API cost)

```
QMD Search Process:
memory_search(query) → QMD sidecar → local embedding → return Top-K results
No external API calls throughout, token consumption = 0
```

#### Measured Data

| Metric | Before | After | Savings |
|--------|--------|-------|---------|
| Heartbeat Detection (per 30min) | ~200 tokens/call cloud | 0 tokens (qwen local) | 100% |
| Security Audit (per 5min) | ~500 tokens/call cloud | 0 tokens (qwen local) | 100% |
| Memory Retrieval | ~1,500 tokens/call (cloud semantic search) | 0 tokens (QMD local) | 100% |

#### Cost Savings
Based On GPT5.4 Nano
```
Heartbeat Detection: 48 calls/day × 200 tokens × $0.73/MT × 30 days = $0.21/month
Security Audit: 288 calls/day × 500 tokens × $0.73/MT × 30 days = $3.20/month
Memory Retrieval: 10 calls/day × 1,500 tokens × $0.73/MT × 30 days = $0.33/month
────────────────────────────────────────────────────────
Total Savings: ~$3.74/month
```

#### Resource Consumption Changes

| Resource | Change |
|----------|--------|
| CPU Usage | ⬆️ Slight increase (qwen 3b + QMD inference) |
| GPU Usage | ⬇️ Significant decrease (high-frequency tasks offline) |
| Network I/O | ⬇️ Fewer API calls |
| Response Speed | ⬆️ Faster local inference (no network latency) |

---

### 4. Direct Script-to-API Calls, Bypassing Bootstrap

#### Optimization Principle

Traditional approach: Have the LLM read workspace context → analyze problem → return result. But many repetitive tasks (like portfolio analysis, market briefs) can **directly call APIs with Python scripts**, bypassing the LLM bootstrap process.

```
Traditional Path (wasting tokens):
cron trigger → LLM load bootstrap(2K tokens) → understand task(500 tokens) → curl API → return(800 tokens)

Optimized Path (zero bootstrap):
cron trigger → Python direct API call → format output → QQ push
Never passes through LLM

or

You use a medium level model like Minimax 2.7 token plan with 10$ monothly. And you ask your agent to rely on Minimax 2.7(Or local LLM). When you need to solve a complicated logic problem or difficult task, you ask your agent on Minimax to take only the necessary text out and send it to openrouter's bigger LLMs, thus bypassing the bootstarp
and the entire history context.
```

**Key Script**: Write a python sciprt such as "ask_openrouter.py",
and put it in your agent's workspace. When you feel the problem is complex and it is better to solve it with bigger and expensive models, you ask your agent to use this script to use openrouter's big models.

```python
# Minimal OpenRouter call — no workspace pollution
# Send pure requests directly, don't load AGENTS/SOUL/MEMORY etc.
```

Similarly, `ask_openrouter_search.py` bypasses the LLM for direct web searches. (Openrouter allows you to add a :online suffix to any model to enable web search)

#### Measured Data

Using "portfolio deep analysis" task as example:

| Path | Tokens per Task | Cost (Sonnet) |
|------|-----------------|----------------|
| Via LLM channel(even without any history) | ~9,000 tokens | $0.081 |
| Python direct call OpenRouter | ~1,200 tokens (API input/output only) | $0.01 |
| **Savings** | **87%** | — |

With 2 analysis tasks daily: **$0.007 × 30 = $0.21/month** (this single task isn't expensive, but the pattern scales to all cron tasks)
If you are using 10 cron to do the news analysis,you save 2.1$/month

#### Resource Consumption Changes

| Resource | Change |
|----------|--------|
| GPU / LLM Inference | ⬇️ Bypass bootstrap, reduce ~3,300 tokens/task |
| Network Calls | → Flat (still need API calls) |
| Maintainability | ⬆️ Python scripts more controllable than prompt engineering |

---

### 5. Console Commands Replace LLM Conversation

#### Optimization Principle

Many operations don't need LLM involvement at all. For example, restarting services, checking status, running known scripts — use shell's `exec` tool directly, no need for the LLM to "understand" and then "act".

PS: This part requires user's intervention

```
User: "Restart openclaw"
LLM method: Load bootstrap → understand intent → generate command → execute → return result
            (~3,000 tokens wasted)

exec method: Directly execute openclaw gateway restart
             (0 tokens)
```

#### Practical Application

| Scenario | LLM Channel | exec Direct | Savings |
|----------|-------------|-------------|---------|
| Restart Service | ~3,000 tokens | 0 tokens | 100% |
| Check Service Status | ~3,500 tokens | 0 tokens | 100% |
| Run Monitoring Script | ~2,000 tokens | 0 tokens | 100% |
| View Logs | ~2,500 tokens | 0 tokens | 100% |

Based on GPT5.4 nano:
With 5 such operations per day as example: **~14,000 tokens × $0.74/MT × 30 = $0.31/month**

#### Resource Consumption Changes

| Resource | Change |
|----------|--------|
| LLM Inference | ⬇️ Zero LLM for daily maintenance |
| Response Speed | ⬆️ Immediate execution (no LLM processing latency) |
| Error Rate | ⬇️ Deterministic execution (no hallucination risk) |

---

### 6. Daily Logic CPU-fication (Python Cron Direct Push)

#### Optimization Principle

If high-frequency scheduled tasks (like market monitoring, price pushes) go through LLM each time, token consumption is astronomical. Correct approach:

```
Python script → fetch data → conditional judgment → push directly via QQ/notification channel
Never goes through LLM, GPU untouched
```

#### Implemented CPU-fied Tasks

| Task | Frequency | Method | Savings |
|------|-----------|--------|---------|
| 📊 Intraday Monitoring | Every 10 min | Python `intraday_watch.py` direct IM push | 100% |
| 🪙 BTC/ETH Monitoring | Every 15 min | Python `price_monitor.py` direct IM push | 100% |
| 🌤️ Airticket Check | Every 2 hours | Python `airticket_monitor.py` direct IM push | 100% |
| 🌡️ Weather Forecast | 2x per day | Python `weather_monitor.py` direct IM push | 100% |
| 🔐 Security Scan | Every 30 min | qwen2.5:3b local scan | 100% |

#### Measured Comparison

If these 5 tasks went through LLM (based on Sonnet):

```
Intraday: 144 calls/day × 1,500 tokens = 216,000 tokens/day
BTC Monitoring: 96 calls/day × 1,200 tokens = 115,200 tokens/day
METAR: 8 calls/day × 1,000 tokens = 8,000 tokens/day
Weather: 2 calls/day × 1,000 tokens = 2,000 tokens/day
Security Scan: 288 calls/day × 500 tokens = 144,000 tokens/day
──────────────────────────────────────────────
Daily: 485,200 tokens → 14.6M/month
Monthly Cost: 14.6M × $9/MT = $131/month
```

**Actual Monthly Cost: $0** (all CPU-fied, zero LLM consumption)

#### Technical Implementation

```python
# intraday_check.py Core Logic
# 1. Fetch market data (LongBridge API)
# 2. Calculate volatility
# 3. Conditional judgment: index >1.2% or stock >2%
# 4. subprocess Popen direct IM push (8s timeout to prevent hang)
# 5. Zero LLM tokens throughout
```

#### Resource Consumption Changes

| Resource | Change |
|----------|--------|
| LLM API Calls | ⬇️ Reduce ~500 calls/day |
| CPU Usage | ⬆️ Python script polling cost |
| Real-time Performance | ⬆️ No need to wait for LLM processing |
| Reliability | ⬆️ Deterministic logic (no hallucinations) |

---

### 7. Intelligent Demands Pulled Back from LLM to CPU (Heartbeat Checklist-ification)

#### Optimization Principle

Heartbeat is OpenClaw's "heartbeat" mechanism — periodically triggers specific tasks. But if heartbeat content is written as vague natural language, the LLM must "understand" it each time.

**Checklist-ification Transformation**: Convert heartbeat prompts into structured execution checklists, run with the lightest model (qwen2.5:3b, locally free), only do simple status confirmation.

#### Transformation Comparison

Before Transformation (traditional heartbeat):
```
"You are a security audit expert, please check system configurations..."
→ LLM (MiniMax) must understand this prompt → inference → execution
→ ~500 tokens/call
```

After Transformation (checklist-ified heartbeat):
```yaml
heartbeat:
  model: ollama/qwen2.5:3b        # Local free model
  lightContext: true               # Don't load full workspace
  prompt: "Execute steps: 1.Read cron file 2.Check key leaks 3.Output HEARTBEAT_OK"
# If output == HEARTBEAT_OK → nothing happens
# If output != HEARTBEAT_OK → push to user
```

#### Measured Data

| Metric | Before | After | Savings |
|--------|--------|-------|---------|
| Heartbeat Model | GPT Nano ($0.75/MT) | qwen local ($0) | 100% |
| Tokens per Step | ~500 | ~200 (but free) | 100% cost |
| Context Mode | full (2K bootstrap) | lightContext (no bootstrap) | Additional 2K savings |
| Security Audit | MiniMax ($0.75/MT) | qwen local ($0) | 100% |

#### Cost Savings
Based on GPT5.4 Nano
```
Heartbeat Detection: 48 calls/day × (200+2000) tokens × $0.74/MT × 30 = $2.34/month
Security Audit: 288 calls/day × (500+2000) tokens × $0.74/MT × 30 = $12.00/month
──────────────────────────────────────────────────────────────
Total Savings: ~$14.34/month
```

> This is just single-agent user savings. Enterprise deployments (multiple agents, multiple workspaces) multiply this by the number of agents.

#### Resource Consumption Changes

| Resource | Change |
|----------|--------|
| API Cost | ⬇️ Two highest-frequency tasks completely offline |
| CPU Usage | ⬆️ qwen2.5:3b continuous running (~3GB RAM) |
| Security | ⬆️ Security audit independent of external APIs (data stays server-side) |
| Response Latency | ⬆️ Faster (local model inference <50ms) |

---

## Comprehensive Benefit Assessment

### Monthly Cost Comparison Summary
Based on mixed use of Sonnet, GPT 5.4 Nano...
| Optimization | Before | After | Monthly Savings |
|--------------|--------|-------|-----------------|
| ① Tree-Structured Documents | $0.41 | $0.14 | $0.27 |
| ② Compaction | $2.66 (no compression long-term session) | $1.33 | $1.33 |
| ③ Local Models (QMD + Ollama) | $3.74 | $0 | $3.74 |
| ④ Direct Script API 10 crons | $2.4 | $0.3 | $2.1 |
| ⑤ exec Replacing LLM | $0.31 | $0 | $0.31 |
| ⑥ CPU-fying cron Tasks | $131 | $0 | $131 |
| ⑦ Heartbeat Checklist-ification | $14.34 | $0 | $14.34 |
| **Total** | **$154.86** | **$1.77** | **$153.09/month** |

> Above is conservative single-user usage estimation. In actual use, conversation patterns vary, savings fluctuate, but trends are consistent.

### Annualized Comparison

| Metric | Before Optimization | After Optimization |
|--------|---------------------|-------------------|
| Annual API Cost | ~$1829 | ~$200 |
| Annual Savings | — | **~$1629** |
| Carbon Reduction (est.) | — | ~95% API calls reduction |

### Beyond Just Saving Money

| Dimension | Benefit |
|-----------|---------|
| ⚡ **Response Speed** | High-frequency tasks from LLM polling → CPU direct push, latency from seconds to milliseconds |
| 🔒 **Privacy & Security** | Memory retrieval and data audit localized, no need to upload to third-party APIs |
| 🛡️ **Stability** | CPU tasks independent of API availability, won't break due to API downtime |
| 📐 **Maintainability** | Rule files modularized, changing one piece doesn't affect others |
| 🧪 **Testability** | Python scripts can be unit tested, LLM prompts rely only on "feelings" |

---

## Appendix 1: Model Pricing Reference

| Model | Provider | Input $/MT | Output $/MT | Average $/MT |
|-------|----------|-----------|-----------|-------------|
| MiniMax M2.7(fixed number monthly if token plan ) | MiniMax API | $0.279 | $1.20 | $0.74 |
| DeepSeek V3.2 | OpenRouter | $0.252 | $0.378 | $0.315 |
| DeepSeek V4 Flash | OpenRouter | $0.112 | $0.224 | $0.168 |
| DeepSeek V4 Pro | OpenRouter | $0.435 | $0.87 | $0.6525 |
| Gemini Flash 3 | OpenRouter | $0.25 | $1.5 | $0.875 |
| GPT-5.4 Nano | OpenRouter | $0.2 | $1.25 | $0.7 |
| GPT-5.5 Pro | OpenRouter | $30 | $180 | $105 |
| Claude Opus 4.7 | OpenRouter | $30 | $150 | $90 |
| Claude Sonnet 4.6 | OpenRouter | $3 | $15 | $9 |
| Grok 4.3 | OpenRouter | $1.25 | $2.5 | $1.875|
| Qwen2.5:3b | Ollama Local | $0 | $0 | $0 |

---

## Appendix 2: Vectorization of Skill Descriptors

If you install too many skills in openclaw, all skill descriptors will appear in your agent's system prompt. You can have your Agent install a RAG module, combined with openclaw message hooks, to intercept messages before sending to LLM, vectorize them, compare with local skill vector chunks, and only pull relevant skill portions into the system prompt. This can save you tens of thousands to hundreds of thousands of tokens.

---

## Conclusion

The core idea of seven optimization techniques can be summarized in one sentence:

> **Transform LLM from "all-purpose butler" to "expert advisor" — CPU-fy daily operations, let complex reasoning go to large models.**

This is not just a cost-saving strategy, but an architectural philosophy: AI Agent intelligence should be layered — high-frequency low-complexity tasks processed locally on CPU, low-frequency high-complexity tasks processed by cloud large models. This ensures both response speed and privacy while making every API dollar count.

---

*This document is based on OpenClaw 2026.4.23 testing. Data compiled on 2026-05-19 with sensitive information anonymized. Model prices based on current announcement at that time.*
