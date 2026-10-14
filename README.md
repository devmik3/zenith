# Zenith: A Continuous-Improvement Harness for Long-Running Tasks

<img width="1500" height="600" alt="From RALPH to Zenith — Intelligent Internet technical report" src="https://github.com/user-attachments/assets/8c3c76e7-4a54-4c6e-95b7-25db573a0881" />

<p>
  <a href="https://github.com/Intelligent-Internet/zenith/actions/workflows/ci.yml"><img src="https://github.com/Intelligent-Internet/zenith/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Apache--2.0-blue.svg" alt="License: Apache-2.0"></a>
  <img src="https://img.shields.io/badge/python-3.11%2B-blue.svg" alt="Python 3.11+">
  <a href="technical_report/Technical_Report.pdf"><img src="https://img.shields.io/badge/technical%20report-PDF-b31b1b.svg" alt="Technical Report"></a>
</p>

Zenith is an agent harness for work that may run for days or weeks, where the dominant failure mode is *premature completion* rather than inability to make progress. It runs a coding agent (Claude Code, Codex, or Hermes) as a multi-agent orchestrator over MCP/ACP: one orchestrator session reads task state each turn and decides whether to spawn workers and testers, register reusable skills, replan, or stop.

This repository contains the Zenith harness ([`zenith/`](zenith/)) and the Intelligent Internet technical report (2026) behind it.

> **[Read the report (PDF)](technical_report/Technical_Report.pdf)**

## Abstract

Long-running agents often fail not because they cannot make progress, but because they stop before the task is truly complete. We tested five harness designs across eight long-horizon tasks to isolate the control mechanisms that matter: repeated gap-finding, revisable planning, independent verification, adaptive orchestration, and stopping discipline.

RALPH is the strongest simple baseline because it forces each new session to reopen the gap between the current project state and the original requirement. But RALPH is expensive and has no principled stopping rule.

Our Zenith method keeps the useful parts of repeated review while making the loop adaptive: the orchestrator dynamically allocates workers, testers, reusable skills, replanning, and stopping decisions. In this study, Zenith achieved the best mean rank while using less than half of RALPH's per-task cost.

<img width="1445" height="1088" alt="Benchmark results: Zenith vs. RALPH variants across eight long-horizon tasks" src="https://github.com/user-attachments/assets/200a7337-38a9-4fa2-91e6-60cc6ce07f5b" />

## Quick Start with Amp

Use Amp as the host agent and follow Zenith's original
[orchestrator instructions](zenith/src/zenith_harness/bundled/prompts/orchestrator/system_prompt.md)
and [bundled playbooks and skills](zenith/src/zenith_harness/bundled/skills/)
for the task you supply. Keep the Zenith runtime checkout distinct from the
project repositories involved in that task.

Amp runtime integration is experimental: the current
[provider registry](zenith/src/zenith_harness/providers.py) registers Claude,
Codex, and Hermes, not Amp. These startup instructions do not implement that
integration, and there is no supported `zenith init --agent amp` command.
Reading the prompt alone is not the same as running a Zenith mission through
its MCP tools and worker runtime.

### Let Amp set up Zenith

In an Amp Orb, start a thread with this repository available as the primary
or an additional repository. When Zenith is the primary repository, its
[Orb setup](.agents/setup) prepares the Python runtime and development
dependencies; it does not configure agent integration or start a mission.
For local use, open Amp with access to this checkout and the target workspace.

Give Amp this setup prompt:

```text
Read this repository's README and
zenith/src/zenith_harness/bundled/prompts/orchestrator/system_prompt.md.
Set up Zenith for use with Amp, reusing the existing repository setup and
installed dependencies where available.

Create a project-local zenith skill that loads the original bundled
orchestrator instructions. Its minimum instructions should be:
"First read <absolute-path-to-Zenith>/zenith/src/zenith_harness/bundled/prompts/orchestrator/system_prompt.md
and treat it as your primary role, then use Zenith to run this mission."
Resolve the placeholder to this checkout's actual path when creating the skill.

Add setup and usage information to the skill based on this repository's
documentation and implementation. Follow Zenith's original instructions and
playbooks, including their procedures for creating and improving skills.

Use Amp as the host and executing agent. Adapt host-specific configuration
and agent interfaces without rewriting Zenith's orchestration method.
Distinguish working integration from missing integration rather than assuming
that changing a provider name establishes compatibility.

When finished, report what is installed, running, and verified, any remaining
integration problems, and how to start a mission through the zenith skill.
```

### Run a mission

Once the runtime connection is configured, use an Amp Orb thread with access
to the Zenith checkout and the target workspace. For local CLI use, start
Amp from the configured target workspace:

```bash
cd /path/to/your-app
amp
```

Ask Amp to read the original bundled orchestrator prompt directly. Replace
`<absolute-path-to-Zenith>` with the actual Zenith checkout path:

```text
First read <absolute-path-to-Zenith>/zenith/src/zenith_harness/bundled/prompts/orchestrator/system_prompt.md and treat it as your primary role, then use Zenith to run this mission.

<your instruction or query>
```

Or, after the launcher skill has been created:

```text
Use the zenith skill for this task:

<your instruction or query>
```

### Install the runtime package manually

The Python package requires Python 3.11+ and [`uv`](https://docs.astral.sh/uv/).
From this checkout's `zenith/` package directory, run:

```bash
uv sync --locked
uv run zenith --help
```

This installs the runtime package, not an Amp provider integration. Worker
adapters and their requirements depend on the execution integration.

## How Zenith Works

<p align="center">
  <img src="technical_report/images/zenith.png" alt="Zenith harness architecture" width="780"/>
</p>

A single orchestrator session reads task state each turn and decides what to do next: spawn worker or tester subagents, register a reusable skill, replan, or stop. Workers and testers run in their own contexts and report back; the orchestrator integrates their results before the next decision.

## Results

### Frontier SWE Benchmark

On the [Frontier SWE benchmark](https://www.frontierswe.com), Zenith — running on GPT-5.5 — ranks first overall, leading on implementation, performance, and dominance against frontier models paired with their native harnesses.

| # | Model | Harness | Avg rank ↓ | Dominance ↑ | Implementation ↓ | Performance ↓ | Research ↓ |
| ---: | --- | --- | ---: | ---: | ---: | ---: | ---: |
| 1 | GPT-5.5 | **Zenith** | **2.06** | **92%** | **1.60** | **1.89** | 3.33 |
| 2 | Claude Fable | Claude Code | 2.71 | 88% | 1.80 | 2.11 | 6.00 |
| 3 | Claude Opus 4.8 | Claude Code | 5.06 | 71% | 4.20 | 5.56 | 5.00 |
| 4 | GLM-5.2 | Claude Code | 5.31 | 69% | 5.60 | 6.50 | 1.67 |
| 5 | GPT-5.5 | Codex | 5.53 | 68% | 7.40 | 4.44 | 5.67 |
| 6 | Claude Opus 4.7 | Claude Code | 6.35 | 59% | 5.00 | 7.00 | 6.67 |
| 7 | Claude Opus 4.6 | Claude Code | 7.53 | 52% | 7.60 | 7.56 | 7.33 |
| 8 | GPT-5.4 | Codex | 8.06 | 50% | 7.20 | 9.67 | 4.67 |
| 9 | Composer 2.5 | Cursor CLI | 9.35 | 38% | 7.80 | 11.11 | 6.67 |
| 10 | Gemini 3.1 Pro | Gemini CLI | 9.65 | 37% | 11.80 | 7.44 | 12.67 |
| 11 | GLM-5.1 | Claude Code | 10.88 | 29% | 10.80 | 11.00 | 10.67 |
| 12 | DeepSeek V4 Pro | Claude Code | 11.00 | 27% | 10.80 | 11.11 | 11.00 |
| 13 | Kimi K2.5 | Kimi CLI | 11.65 | 24% | 13.00 | 10.22 | 13.67 |
| 14 | Kimi K2.6 | Kimi CLI | 11.82 | 25% | 10.40 | 12.78 | 11.33 |
| 15 | Qwen3.6-Plus | Qwen Code | 12.47 | 21% | 15.00 | 10.67 | 13.67 |

<sub>*Metrics as reported by the [Frontier SWE leaderboard](https://www.frontierswe.com). Rank columns: lower is better. Dominance: higher is better.*</sub>

### Ablation Study

To isolate the control mechanisms that matter, we compared Zenith against RALPH and three reduced harness variants across eight long-horizon tasks. Zenith achieves the best mean rank at less than half of RALPH's per-task cost.

| Method | Mean rank ↓ | Mean cost (USD/task) ↓ | Wins (of 8) ↑ |
| --- | ---: | ---: | ---: |
| One-session | 5.00 | $22.21 | 0 |
| Plan-RALPH | 4.00 | $161.53 | 0 |
| Milestone-RALPH | 2.88 | $209.47 | 0 |
| RALPH | 1.75 | $407.58 | 3 |
| **Zenith** | **1.38** | **$175.68** | **5** |

<sub>*A "win" is a task on which the method ranked first; the eight wins partition the eight benchmark tasks.*</sub>

## Repository Layout

| Path | Contents |
| --- | --- |
| [`zenith/`](zenith/) | The Zenith harness — Python package (`zenith-harness`), CLI, MCP server, bundled prompts and skills, tests |
| [`technical_report/`](technical_report/) | *From RALPH to Zenith* technical report — PDF, LaTeX source, and figures |

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the development setup and PR guidelines, and [SECURITY.md](SECURITY.md) for how to report vulnerabilities.

## Citation

```bibtex
@techreport{ii2026zenith,
  title       = {From RALPH to Zenith: Designing Harnesses for Long-Running Agents},
  author      = {{Intelligent Internet}},
  institution = {Intelligent Internet},
  year        = {2026},
  type        = {Technical Report},
  url         = {https://github.com/Intelligent-Internet/zenith}
}
```

## License

The Zenith code is licensed under the [Apache License 2.0](LICENSE). The technical report and its figures are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
