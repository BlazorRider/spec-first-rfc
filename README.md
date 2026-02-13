[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![RFC Status: Draft](https://img.shields.io/badge/RFC-Draft-yellow.svg)](RFC.md)

# Spec-First Development with MCP-Powered Gap Detection

**An RFC documenting a methodology for keeping architectural intent and production code aligned — using LLMs and MCP as the bridge.**

---

## The Problem

Enterprise projects don't fail because developers write bad code. They fail because **spec and code drift apart** — silently, across hundreds of files, over months and years.

Every generation of enterprise tooling has tried to solve this:
- **Process modeling** (ARIS, €500k+/year) formalized the spec but couldn't reach the code
- **Model-based test automation** (TOSCA, €100k+/year) built testability into models but required expensive platforms
- **Static analysis** (€30k+/year) checked code but couldn't understand architectural intent

The missing piece was always the same: something that understands both the spec (what we intended) and the code (what we actually built) — and can detect the gap.

## The Approach

**LLMs + MCP close the loop.** A Model Context Protocol server pre-analyzes the codebase with Roslyn, indexes versioned framework documentation, and provides embedding-powered semantic search. AI assistants query structured knowledge instead of traversing files — and continuously validate that generated code matches the project's architectural intent.

This methodology has been developed and validated in a production government digitalization project: 170k+ lines of code, C#/.NET 10, Blazor Server, ABP.io, Domain-Driven Design, multiple modules with explicit boundaries.

## What's in the RFC

The full RFC documents:

1. **Why spec-code drift happens** — the brownfield context gap, framework versioning, convention decay
2. **Architecture** — MCP knowledge server with Roslyn analysis, framework indexes, semantic search
3. **Five patterns** — Knowledge Server, Worker Prompts, Knowledge Base, Work Item Integration, Scaffold-Validate-Test Pipeline
4. **Results** — observed metrics from production use (what improved and what didn't)
5. **Maturity assessment** — honest evaluation of each component
6. **Lessons learned** — including known risks and open questions

**[Read the full RFC →](RFC.md)**

---

## Design Tenets

- **Human-in-the-loop by default.** No automated writes, no autonomous chaining.
- **Deterministic workers over generative freedom.** Decision trees over open-ended prompts.
- **Project knowledge over generic LLM assumptions.** The server knows *this* codebase.
- **Validation before generation.** Check conventions first, then scaffold.

---

## This is a Practitioner Report

This RFC describes patterns, not products. The internal tooling is project-specific and not publicly available. The methodology is shared so others can adapt it for their own projects.

---

## Author

**[BlazorRider](https://github.com/BlazorRider)** — Enterprise software architect, 25+ years. Currently building spec-first development methodology in a government enterprise project.

[Twitter/X: @BlazorRider](https://twitter.com/BlazorRider)

---

## Feedback

Issues and pull requests are welcome. This RFC targets:
- Enterprise architects evaluating AI-assisted development
- MCP server builders
- .NET / DDD / ABP.io teams in large codebases
- Anyone who's watched spec and code drift apart
