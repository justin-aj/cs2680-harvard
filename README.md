# CS2680 Slide Summaries

Personal study notes summarizing the lecture slides for Harvard CS2680, *Modern AI Systems: Agents and System Optimizations* (instructor: Juncheng Yang).

## Course links

- Course website: https://cs2680.com
- Alternate course domain (shown in the slide footers): https://course.agentic-system.org
- Agent evaluation notes: https://cs2680.com/notes/agent-evaluation

## Slides

The original slide PDFs are included in this repository with the instructor's permission.

| Lecture | PDF |
|---|---|
| 1. Intro | [1_intro.pdf](1_intro.pdf) |
| 2. Agent From a User's Perspective I | [2_agent_use_1.pdf](2_agent_use_1.pdf) |
| 3. Agent From a User's Perspective II | [3_agent_use_2.pdf](3_agent_use_2.pdf) |
| 4. Agent Design I | [4_agent_design_1.pdf](4_agent_design_1.pdf) |
| 5. Agent Design II | [5_agent_design_2.pdf](5_agent_design_2.pdf) |
| 6. Agent Design III | [6_agent_design_3.pdf](6_agent_design_3.pdf) |
| 7. Agent Design IV | [7_agent_design_4.pdf](7_agent_design_4.pdf) |
| 8. Agent Design V | [8_agent_design_5.pdf](8_agent_design_5.pdf) |
| Assignment 3 | [8_assignment3.pdf](8_assignment3.pdf) |

## 1. Intro (50 slides)

- Course overview: foundations (LLMs, Claude Code, coding agents), then systems (GPU kernels, batching and scheduling, KV/prefix cache, routing, agent serving).
- Five assignments: (1) web UI for Claude Code, (2) a simple coding agent, (3) optimize the agent on a leaderboard, (4) serve the agent on a small model, (5) full-stack optimization.
- Grading: assignments 70%, project 16%, presentation 10%, participation 4%, bonus 10%. Late penalty is 20% per day, with 4 late days.
- Policy: AI use is encouraged but must be disclosed, and session logs are required. In-person attendance. Compute is a class HPC cluster (RTX 6000 Blackwell, SLURM) plus AWS.
- Part I, how AI systems differ from traditional systems: heterogeneous compute, prefill (compute-bound) vs decode (memory-bound), floating-point non-determinism, huge state (weights, KV cache), multi-GPU communication, reliability, power.
- Part II, agents: plan, act, observe, adapt. The "agent as a new computer" analogy (LLM = CPU, context = RAM, tools = peripherals, harness = OS). Why agents are a systems problem. The agentic stack. The real objective is $ per successful task, not tokens per second.
- Out of scope: training and model architecture.

## 2. Agent From a User's Perspective I: How Claude Code Works (53 slides)

- The agentic loop: gather context, the model proposes an action, the harness executes it, and the result folds back into context.
- More than 10K tokens are in context before the first keystroke. The repository is pulled in on demand with grep and read, not sent whole.
- Permission modes, context growth, and compaction.
- Memory: the CLAUDE.md hierarchy and what belongs in it, `.claude/rules/` with `paths:` frontmatter, auto memory with a MEMORY.md index, and the `/memory`, `/context`, `/init` and `/doctor` commands.
- Core tools: Read, Search, Fetch, Edit/Write, Shell, Delegate.
- Extensions: a Skill defines how to do a task, MCP provides external access, a Hook enforces behavior at lifecycle events, and a Plugin packages all of them. Includes a release-workflow example and a classification exercise.
- Optional slides: MCP architecture, its primitives (tools, resources, prompts), security, gateways, and the 2026-07-28 spec.

## 3. Agent From a User's Perspective II: Multi-agent and Best Practice (72 slides)

- Case studies: the Superpowers plugin (nine skills plus a SessionStart bootstrap hook) and the Caveman plugin (skill, commands, hooks, MCP, and a local proxy that compresses tool output).
- Scaling: parallel agents with git worktrees, subagents (built-in, custom, nested), agent teams, dynamic workflows, `/goal` and `/loop`.
- Case study: sixteen agents built a C compiler for about $20K, coordinated through Git.
- "Programming 3.0" and Karpathy's Software 3.0. Uncertainty is the core theme, so make the surrounding system dependable.
- Codex usage reports (June and September 2026): delegated tasks keep growing, one human runs many agents, and the delegation ladder.
- Optional slides: a survey of CLI coding agents.

## 4. Agent Design I: Context Management and Tool Design (56 slides)

- The ReAct loop and the agent loop in code.
- Search tools compared: grep/glob, semantic index, extrinsic signals, and LSP.
- Edit tools compared: whole-file rewrite, unified diff, search/replace, and AST edits.
- The risks of a shell tool, and tool design best practices.
- Prompt engineering: zero-shot, few-shot, role, chain-of-thought, tree-of-thought, self-consistency, prompt chaining, meta prompting.
- Context engineering: context is a finite resource. Four ways to control it: admission, offloading, transformation, compaction. Five ways it fails: poisoning, distraction, confusion, clash, drift.
- Agent = Model × Harness: changing only the harness roughly doubled SWE-bench accuracy.

## 5. Agent Design II: Multi-Agent Systems (46 slides)

- Three graphs behind every design: control, task, and communication.
- Eight architectures: supervisor-worker, hierarchical, pipeline/DAG, blackboard, committee, market, swarm, dynamic topology. Real systems are hybrids.
- Task decomposition: what to split on, the granularity tradeoff, static vs dynamic decomposition, and common failures (overlap, gaps, hidden dependencies, stragglers).
- Communication: medium, channel, and message format (structured control fields plus natural-language content).
- Multi-agent systems inherit distributed-systems problems such as idempotency and partial failure.
- The A2A protocol (agent cards, tasks, task lifecycle) compared with MCP.
- Design principles: a single agent is often enough.

## 6. Agent Design III: Memory and Reliability (31 slides)

- Memory pipeline: write or skip, encode, store, retrieve, reflect, forget.
- Formats (raw episodes, summaries, structured state, knowledge graphs) and backends (files, key-value, vector, graph databases).
- Valid time and provenance, memory ownership, and dedicated memory systems (Mem0, Graphiti, supermemory).
- Memory failure modes and benchmarks (LoCoMo, LongMemEval, BEAM).
- Reliability: at 95% success per step, a 50-step run finishes without error only about 8% of the time.
- Fault origins: task specification, harness, model, tool, environment. Each has a failure-and-fix table, followed by a "design for failure" summary.

## 7. Agent Design IV: Evaluation (37 slides)

- Component-level vs end-to-end evaluation, benchmark history, and metrics (pass@k vs pass^k, cost, latency).
- Benchmark deep dives: GAIA, BFCL, WebArena, OSWorld, SWE-bench, τ-bench, Terminal-Bench, TheAgentCompany.
- Challenges: data leakage across the whole agent stack, single-turn bias, non-determinism, unpenalized side effects, and fast benchmark saturation.
- Assignment 3 is scored on a Pareto skyline of capability, cost, and speed.

## 8. Agent Design V: Cost-Effective Agents (54 slides)

- Cost = users × sessions × turns × steps × tokens × price per token.
- Class usage data: about 97.6% of tokens are cache reads, and the median main-agent step is 200K–300K tokens.
- Levers: fewer turns, batched steps, leaner context, shorter output (effort levels), and a lower price per token (caching, batching, routing).
- Budgets and stop rules, and when not to use an LLM at all.
- Speed: the critical path, orchestration, parallelism, and speculative actions (PASTE, SPORK).
- Harness architectures: monolithic, modular, microkernel.
- Eight harnesses compared: Claude Code, Codex CLI, DeepSeek Harness, OpenCode, Pi, oh-my-pi, OpenClaw, Hermes Agent.

## Assignment 3: MadsOpt Harness Competition (7 slides)

- 26 tasks across 3 container types. The agent reaches each task over HTTP and runs offline except for the course API.
- Limits: three model tiers (Expert, Standard, Starter), 35 minutes per `evaluate()` and 4 hours in total, $10 per run and $30 per day.
- Only `src.zip` is uploaded to the leaderboard.
- Ranking is a skyline over tasks solved, cost per solved task, and time per solved task. At least 7 tasks must be solved, and the baseline is 16 tasks.
- Score: `max(81 − r, 50)` if any submission reaches the baseline, otherwise `81 − r`. The leaderboard is worth 80 points, the writeup 10, and the video 10.
