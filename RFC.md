# RFC: Spec-First Development with MCP-Powered Gap Detection

**A Pattern for Measuring the Distance Between Specification and Code**

*Author: BlazorRider — Enterprise Software Architect*
*Date: February 2026*
*Status: Draft for Community Review*

---

## Abstract

Projects fail because specification and code drift apart — and nobody notices until it's too late. This RFC describes a pattern that makes this drift measurable and automated: a spec-first methodology where machine-readable architecture documentation serves as the single source of truth, validated continuously against the codebase using MCP (Model Context Protocol) servers and specialized worker prompts.

The pattern has been developed in 2025 and validated since January 2026 in a government digitalization project built with ABP.io, Blazor Server, and DDD principles. First measurements show a 90% reduction in context window consumption compared to file-read approaches, and automated gap detection across 6 modules in under 54 minutes — a task that previously required 2–3 days of manual review.

A native MCP orchestrator — built to bypass the LLM overhead for deterministic checks — completes a full sprint check in 532ms with 50 MCP calls. The same check through an LLM takes approximately 8 minutes. This validates the core hypothesis: when the specification is formal enough, most quality checks don't need AI — they need structured data and decision trees.

This is not a product announcement. It's a practitioner report on what works, what doesn't, and what we learned.

---

## 1. Why Do Projects Fail?

Not because of bad developers. Not because of wrong technology. Not because of missing methodology.

Projects fail because specification and code drift apart — and nobody notices until it's too late.

This observation isn't new. It's the same problem that enterprise tooling has been trying to solve for decades. What's new is that LLMs, combined with structured knowledge servers (MCP), can now detect this drift automatically — if the specification is formal enough for a machine to read.

### 1.1 The Spec-in-a-Drawer Problem

Most specifications share a common fate: they're written before development, approved in a review meeting, saved as a PDF, and never opened again. By sprint 3, the code has diverged. By sprint 10, the specification is fiction.

This isn't negligence. It's a structural problem: maintaining a specification alongside code requires effort that delivers no immediate value to the sprint. So it doesn't happen.

The hypothesis behind this pattern: **What if the specification IS the development environment?** What if the same document that describes the architecture also drives code generation, test derivation, and architecture validation — so that keeping it current is not extra work but the primary workflow?

### 1.2 Three Predecessors — Same Principle, Three Eras

The idea that formal models can drive machine-derived outputs isn't new. Three enterprise tools taught the same lesson across different decades:

**Process Modeling (2000s):** Tools like ARIS demonstrated that if your business process model is formal enough (level 3+4), a machine can derive code from it. The lesson: formal models find gaps that informal descriptions miss. The cost: €500k+/year in licensing.

**Test Automation (2010s):** Tools like TOSCA showed that test cases can be derived from formal process descriptions. If your model describes every state transition, a machine can generate test steps. The lesson: testability must be built into the model, not added afterward. The cost: €100k+/year in licensing, plus specialists at €1,200+/day.

**Static Code Analysis (2010s–present):** Coding conventions and architecture rules, enforced by static analysis tools with management reporting. The lesson: governance without tooling is wishful thinking. The cost: €30k+/year.

**The common principle:** Formal models → machine derivation → automated quality assurance. Three tools. Three eras. One idea. Combined licensing: €630k+/year.

### 1.3 The 2025 Question

What if an LLM can do all three?

If your specification is formal enough — machine-readable Markdown with exact state machines, API contracts, validation rules, and role definitions — then an LLM can:

- **Derive code** from it (like process modeling tools)
- **Derive tests** from it (like test automation tools)
- **Find architecture violations** in existing code (like static analysis)
- **Find gaps between spec and code** — something none of the three could do alone

Not because the LLM is smarter than those tools. But because it can read both the specification AND the code in the same context window, and measure the distance between them.

---

## 2. The Spec-First Method

### 2.1 A Specification That's Never Finished

The core idea: the specification is not a document you write once and file away. It's the **single source of truth** across the entire product lifecycle — a living document that the gap detector checks every sprint against the actual code.

This means the specification is deliberately ahead of the code. In early sprints, most of it describes functionality that hasn't been built yet. That's not a bug — it's the method. The spec-code delta IS the backlog, measured and prioritized automatically.

The specification is written in Markdown — not because it's trendy, but because it serves five consumers from a single source:

- **LLM workers** read Markdown directly for code generation
- **MCP servers** index it for structured tool calls
- **Gap detectors** compare it against the codebase
- **Static site generators** produce a navigable website for stakeholders (with search, rendered diagrams, dark mode)
- **Diagram renderers** produce PNGs for workshops and reports

One file. Five consumers. No synchronization problem.

### 2.2 Design Principles

**XOR-only gateways.** No OR, no AND. Why? Because OR gateways create combinatorial explosions. 5 OR gateways × 3 paths = 243 combinations. C1 branch coverage becomes unaffordable. XOR: 5 × 3 = 15 test cases. 100% C1 coverage. This applies to state machines, confidence gates, and process flow diagrams.

**Confidence Gates.** Every architecture chapter defines two levels for AI workers:
- 🟢 CONFIDENT → AI works autonomously. The condition is explicitly defined.
- 🔴 UNCERTAIN → Stop and ask the human. No guessing.

This is QA thinking applied to AI copilots. The gates are essentially test specifications — expected behavior (green) and forbidden behavior (red).

**Anti-hallucination markers.** Every chapter includes explicit statements: *"Do NOT hallucinate: [specific topic]. The relevant information is in Chapter [X]."* This sounds trivial. It measurably reduces incorrect code generation because it gives the LLM a concrete alternative instead of silence.

**Deterministic decision trees.** Workers follow decision trees, not natural language instructions. Every branch is a XOR decision. Every leaf is a concrete action. No ambiguity.

### 2.3 MCP as Knowledge Server

The specification alone isn't enough. An LLM reading 80,000 lines of Markdown and 90,000 lines of code would exhaust its context window before finishing a single module.

MCP (Model Context Protocol) solves this by placing a structured knowledge server between the LLM and the data. Instead of reading files, the LLM calls tools that return pre-processed, structured results.

The current setup uses a multi-server composition:

- **Index Server** — Project structure, entity metadata, architecture rules, framework documentation, specification search
- **Code Analysis Server** — Roslyn AST analysis, layer validation, dependency graphs, naming convention checks
- **Work Item Server** — Sprint scope, work item hierarchy, tag analysis (via cached project management data)

41 tools across the server composition. The key insight: the server doesn't just retrieve data — it processes it. An architecture validation call returns 920 characters of structured violations. The equivalent file-read approach returns 25,550 characters of raw source code that the LLM must interpret on its own.

---

## 3. The Spec-to-Code Pipeline

A single entity specification produces ~20 files across 7 architectural layers, totaling approximately 1,600 lines of code. The pipeline uses 6 specialized workers and ~8 MCP tool calls.

### 3.1 Pipeline Architecture

```
Specification (Markdown)
    ↓
[Domain Worker] → Entity, Constants, Repository Interface
    ↓ (BLOCKING — all others wait for domain model)
[API Worker] → DTOs, AppService, Permissions, Mapper      ┐
[Migration Worker] → EF Config, DbContext, Migration       │ parallel
[Wireframe Worker] → SVG wireframe, Screen Index           ┘
    ↓
[UI Worker] → Razor Pages, Menu  (waits for API + Wireframe)
    ↓
[Seeding Worker] → Test Data
    ↓
[Test Worker] → Unit + Integration Tests
    ↓
[Review Worker] → Architecture Compliance Check
```

The pipeline is deterministic: Domain blocks everything because all other layers depend on entity names, property types, and constants. API and Migration can run in parallel because they don't depend on each other. UI waits for both API contracts (to know which endpoints to call) and wireframe definitions (to know how to render).

### 3.2 Wireframes Without a Designer

Wireframes are generated from ASCII specifications in the architecture documentation, producing SVG files that map to Blazor components via a component mapping table. ~20 SVG wireframes exist, covering the application's screen inventory.

This is unidirectional — spec to wireframe to code. There is no designer roundtrip. The visual quality is functional, not polished. For a government application where UX consistency matters more than visual flair, this is sufficient. For consumer-facing applications, a designer would need to be added to the loop.

### 3.3 Automation Level

~80% of the entity pipeline is automated. The remaining 20% is manual:

- **Merge decisions** — the developer reviews and merges generated code
- **Specification updates** — when gaps are found, the spec is updated manually
- **Architecture decisions** — cross-module concerns require human judgment

The gap between 80% and 100% is deliberate. Full automation without human review would violate the human-in-the-loop principle that makes the pattern trustworthy in regulated environments.

---

## 4. Gap Detection — Measuring the Distance

### 4.1 What It Does

The gap detector is an autonomous agent that reads both the specification and the codebase (via MCP), compares them systematically, and produces:

- A **gap report** per module with typed, prioritized findings
- **Domain flow diagrams** (Mermaid) showing process flows with gaps highlighted in red
- **Workshop-ready questions** for each gap, with decision options and consequences

### 4.2 First Measurement

We ran the gap detector across all 6 modules. Results:

- **Duration:** ~54 minutes for 5 modules (one module was analyzed in a separate session)
- **MCP calls:** ~80 structured tool invocations
- **Context budget:** 26% of the 200,000-token window
- **Gaps found:** 33 total (0 blockers, 22 significant, 11 improvements)
- **Domain flow diagrams:** 10 generated
- **Manual equivalent:** 2–3 days of architect review
- **Estimated false positive rate:** ~10%

### 4.3 Gap Type Distribution

| Gap Type | Count | % | Description |
|----------|-------|---|-------------|
| Spec-Code Delta | 19 | 58% | Spec describes planned functionality not yet implemented |
| Missing API/Permission | 6 | 18% | Screen flow references undefined backend endpoint |
| Data Flow Gap | 4 | 12% | DTO fields without corresponding entity properties |
| Multi-Tenancy Gap | 4 | 12% | Entities missing tenant isolation |

### 4.4 Interpreting the 58%

The dominant gap type requires context to interpret correctly.

In our spec-first approach, the specification is written before the code and is deliberately ahead. The 58% spec-code delta represents planned functionality — it IS the backlog, measured and categorized automatically.

In a mature brownfield project where the specification was written years ago, the same 58% would be a red flag: dangerous drift. Architecture decisions that were documented but never reached the code.

**Same tool. Same measurement. Two interpretations.** The gap detector doesn't judge — it measures the distance between what was specified and what exists. Whether that distance is "planned work ahead" or "dangerous drift behind" depends on where you are in the project lifecycle.

This dual-mode capability is what distinguishes it from both spec generators (which only look at specs) and static analysis tools (which only look at code). Measuring the distance requires simultaneous knowledge of both.

### 4.5 Gap Taxonomy

The detector uses a taxonomy of 12 gap types covering state machine completeness, permission consistency, API coverage, data flow integrity, multi-tenancy compliance, test coverage, and documentation completeness. Each gap is classified by type and priority (P1 blocker through P4 cosmetic).

---

## 5. MCP Performance — Enabler, Not Optimization

### 5.1 Measurement Setup

We measured MCP server performance across 30 individual tool calls (10 tools × 3 runs) using UTC timestamp bracketing around each invocation. Both MCP and equivalent file-read approaches were measured in the same environment.

### 5.2 Server Performance

| Metric | Value |
|--------|-------|
| Tools available | 41 |
| Server-side processing | <30ms |
| End-to-end latency (median) | ~6s |
| Client overhead | ~91% of total latency |
| Response size range | 170–14,000 characters |
| Response determinism | 100% identical across 3 runs |

The server-side processing is negligible. The bottleneck is the LLM tool-invocation framework — JSON-RPC marshalling, session management, response deserialization. This overhead is identical regardless of payload size, which means the architecture scales. A native MCP client without LLM overhead would be approximately 200× faster.

### 5.3 MCP vs. File-Read

| Metric | MCP | File-Read | Improvement |
|--------|-----|-----------|-------------|
| Tool calls (5 tasks) | 6 | 25 | 76% fewer |
| Context consumed | ~5,300 chars | ~54,000 chars | 90% less |
| Estimated tokens | ~1,300 | ~13,500 | 90% less |
| Wall-clock time | ~38s | ~81s | 53% faster |

The token reduction isn't the headline. The headline is what it enables:

| Scenario | MCP Tokens | File-Read Tokens | Verdict |
|----------|-----------|-----------------|---------|
| 1 module analysis | ~9,000 | ~89,000 | File-read uses 45% of budget |
| 5-module gap detection | ~53,000 | ~525,000 | **File-read exceeds 200k budget** |

A 6-module gap detection in a single LLM session is **physically impossible** without MCP. File-reads would exhaust the context window after approximately 2 modules.

### 5.4 The Honest ROI

| Cost (Claude Sonnet) | MCP | File-Read |
|---------------------|-----|-----------|
| Per module analysis | $0.63 | $1.02 |
| 5-module gap detection | $3.13 | $6.84 |
| Annual (sprint-cadence) | ~$80 | ~$180 |

The cost savings are ~$100/year. That's not the point.

MCP doesn't make it cheaper. It makes it possible. An analysis that without MCP doesn't fit into a context window. That's the difference between optimization and enabler.

### 5.5 The Native Orchestrator — Validating the 200× Prediction

Section 5.2 predicted that a native MCP client would be approximately 200× faster. We built one to test that hypothesis.

The orchestrator is a C# console application that calls MCP tools directly via HTTP — no LLM in the loop. It executes deterministic checks (entity completeness, layer violations, state machine validation, multi-tenancy rules) using XOR decision trees derived from the specification.

**Measured results:**

| Scenario | MCP Calls | Duration | LLM Equivalent | Speedup |
|----------|-----------|----------|----------------|---------|
| Full sprint check (12 entities) | 50 | 532ms | ~8 min | ~900× |
| Sprint optimization report | 62 | 584ms | ~10 min | ~1,000× |
| Incremental check (1 module, 4 entities) | 4 | 192ms | ~72s | ~375× |
| Full sprint check (regression) | 50 | 436ms | ~8 min | ~1,100× |

The speedup exceeded the prediction. The reason: the 200× estimate was based on server latency alone (<30ms vs. ~6s). The actual improvement is larger because the orchestrator also eliminates prompt construction, response parsing, and session management overhead that the LLM framework adds.

**What it found on first run:**
- 4 entities missing from codebase (correctly identified as not-yet-implemented)
- 4 multi-tenancy rule violations (entities missing tenant isolation — confirmed real findings)
- 3 entities with incomplete application layer (application services not yet generated)

**What it doesn't replace:** The orchestrator handles ~45% of all checks natively. The remaining 55% require domain understanding, architecture judgment, or prose interpretation — those stay with the LLM. The orchestrator prepares structured prompts for these cases, reducing LLM context consumption.

**Tool classification (41 tools):**

| Category | Count | % | Description |
|----------|-------|---|-------------|
| Native (deterministic) | 28 | 68% | JSON output, C# parseable, no interpretation needed |
| Hybrid (structured + context) | 5 | 12% | Structure parseable, context improves results |
| LLM-required | 8 | 20% | Prose/Markdown, requires comprehension |

**Architecture rule classification (127 rules from specification):**

| Category | Count | % |
|----------|-------|---|
| Natively checkable | 64 | 50% |
| Partially checkable | 38 | 30% |
| LLM-required | 25 | 20% |

The orchestrator runs as a file-watching daemon during development — detecting code changes, identifying affected modules, and running incremental checks in under 200ms. This turns gap detection from a periodic audit into continuous feedback.

---

## 6. What Didn't Work

These limitations are not footnotes — they're the most important section of this RFC. Anyone can describe what works. Describing what doesn't is what makes a pattern trustworthy.

### 6.1 Resolved

**Roslyn MCP server session instability** — Streamable HTTP transport lost sessions during extended analysis runs. This prevented Roslyn AST analysis (dependency graphs, circular references) from being used in the gap detector. *Resolved February 2026.* The full multi-server composition is now operational.

**Orchestrator automation** — The previous version of this RFC reported the orchestrator at 45% maturity with "minimal automation (15%)". Workers were invoked manually via slash commands, not chained automatically. *Resolved February 2026.* A native C# orchestrator now executes deterministic checks autonomously, with 86 passing tests and sub-second sprint checks. The orchestrator is a command-line tool, not a multi-agent system — but it automates the mechanical 45% that previously required an LLM.

**Gap detection was not on sprint cadence** — The previous version noted that tools and taxonomy existed but systematic execution was not implemented. *Resolved February 2026.* The orchestrator's daemon mode watches for file changes and runs incremental checks continuously. Full sprint checks can be triggered on demand in under a second.

### 6.2 Open Limitations

**The spec-to-code loop is not fully closed.** The forward path (spec → code) is automated at ~80%. The reverse path (gap → spec update) is manual. When the gap detector finds that code doesn't match the spec, a human must decide whether to update the code or update the spec. Automating this decision is the next major milestone.

**No designer in the loop.** Wireframes are AI-generated SVGs from ASCII specifications. The visual quality is functional but not polished. There is no designer review, no Figma integration, no visual QA process.

**ADO Work Item signaling is inconsistent.** Only 2 of 6 gap detection pilots created work items automatically. The remaining 4 required manual intervention. The orchestrator improves tag detection and hygiene checking, but the creation loop is not yet zero-touch.

**Sample size is statistically thin.** 3 runs per tool is minimal. The low variance (<20% spread) justifies it for a first measurement, but larger studies are needed before making general claims.

**55% of checks still require LLM.** The orchestrator handles the deterministic half. Domain logic validation, architecture judgment calls, and cross-module semantic analysis still require an LLM. The orchestrator prepares structured prompts for these — but it doesn't replace them.

**Specification completeness varies by module.** Some modules have detailed specifications with exact entity definitions. Others have only high-level descriptions. The orchestrator's effectiveness directly depends on specification quality — garbage in, garbage out.

---

## 7. Differentiation

This pattern occupies a different space than existing approaches:

| Aspect | Spec Generators (OpenSpec, Kiro) | This Pattern |
|--------|--------------------------------|--------------|
| **Primary question** | "What should we build?" | "Does what we built match what we decided?" |
| **Strength** | Greenfield — new projects | Spec-first — AND brownfield maintenance |
| **Input** | LLM knowledge + user requirements | Formal specification + existing codebase |
| **Output** | Specification documents | Gap reports, code generation, test derivation |
| **Validation** | Spec completeness | Spec-code distance measurement |
| **Data source** | LLM training data | Roslyn AST + framework indexes + embeddings |
| **Evidence** | Demos, tutorials | Measured in a production project |

Spec generators solve a real problem: getting from idea to specification. This pattern solves a different problem: keeping specification and code synchronized over the lifetime of a project.

They're complementary, not competitive. A spec generator could produce the initial specification. This pattern would then measure whether the code stays aligned with it — every sprint, automatically.

---

## 8. Outlook

Three areas of active development:

**Closing the loop.** The forward path (spec → code) works. The reverse path (gap → spec update) is manual. The orchestrator now generates structured workshop prompts with decision options and consequences — moving closer to automating the "update spec or update code?" decision through human-in-the-loop workflows.

**Cross-sprint trend analysis.** The orchestrator produces JSON reports with entity scores per sprint. Tracking these scores over time answers the question: is the spec-code distance growing or shrinking? This turns a point-in-time audit into a continuous health metric.

**Cross-project applicability.** The pattern was developed for ABP.io/Blazor/DDD. The principles (formal specs, MCP knowledge server, gap detection, native orchestration) should transfer to other stacks. The orchestrator's rule engine is designed for extensibility — new rules are C# classes implementing a single interface. Testing this hypothesis requires validation in a different technology context.

---

## Appendix A: Maturity Assessment

An honest self-assessment as of February 2026:

| Component | Maturity | Notes |
|-----------|----------|-------|
| MCP Server (index) | 85/100 | 34 tools, stable, production use |
| MCP Server (Roslyn) | 75/100 | Session bug fixed, integrated into orchestrator |
| Knowledge Base | 90/100 | Strongest component — ~180 Markdown files, semantic search |
| Spec-to-Code Workers | 75/100 | ~20 workers, 12 production-ready |
| Gap Detection (LLM) | 70/100 | Tools, taxonomy, and workshop prompts operational |
| Native Orchestrator | 80/100 | 4 phases complete, 86 tests, sub-second checks, daemon mode |
| Wireframe Pipeline | 60/100 | Functional but no designer review |

The orchestrator moves the system from "excellent prompt engineering framework" toward "hybrid architecture" — deterministic checks execute natively, creative work stays with the LLM. It is still not an autonomous multi-agent system.

---

## Appendix B: Methodology

All measurements were taken in February 2026.

**MCP response times:** PowerShell UTC timestamps before and after each tool call. Measured times include Claude Code overhead (~91% of total). Server-side processing is self-reported at <30ms.

**Orchestrator measurements:** Built-in CLI timer measuring wall-clock time from command start to report output. MCP call count tracked internally. All measurements taken against production MCP servers with the production codebase.

**Token estimates:** Character count ÷ 4. This is an approximation; mixed German/English/code content may have a factor between 3.5–4.5.

**Gap detection timing:** Git commit timestamps as proxy for module start/end. Actual analysis time may be shorter (commit preparation overhead not deducted).

**Cost calculations:** Based on Claude Sonnet pricing ($3/M input tokens, $15/M output tokens) as of February 2026.

**Reproducibility:** All measurements can be reproduced with the same MCP server version, codebase revision, and measurement method. The orchestrator's test suite (86 tests) provides regression coverage.

---

## About the Author

Enterprise software architect with 25+ years of experience in large-scale IT projects, including work with major European financial institutions, leading logistics corporations, and German federal agencies.

Currently building enterprise software for a German federal agency using .NET, Blazor Server, ABP.io, and DDD. Background in enterprise architecture (TOGAF), quality management (CMMI), test automation, and process optimization.

This RFC reflects practical experience from a production project — not theoretical patterns from a lab environment.

GitHub: [github.com/BlazorRider](https://github.com/BlazorRider)
