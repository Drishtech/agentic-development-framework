# Agentic Development Framework (ADF)

An LLM-agnostic framework designed to structure software development executed by autonomous agents (especially 8B-32B models). Its goal is to guarantee production-ready code through strict context control, forced TDD, and mechanical validation, minimizing human intervention.

## 🎯 Objective

Current LLM agents suffer from short-term memory, context drift, and a tendency towards technical laziness (avoiding refactoring and tests).

ADF solves this by serializing human intent into rigidly structured text artifacts. It does not rely on the agent's "goodwill," but on **mechanical and executable gates** (`scripts/verify_sprint.py`) that block progress if the agent fails to provide evidence of TDD (Red/Green) or violates architecture invariants.

## 🏗️ Inspiration and Theoretical Foundations

This framework does not emerge from a vacuum. It takes traditional software engineering patterns and adapts them to the cognitive limitations of LLMs:

1. **[Diátaxis](https://diataxis.fr/)**: Adopts strict separation of documentation by purpose, eliminating unnecessary tutorials to force the agent to operate solely with reference manuals (Reference/How-to).
2. **[Architecture Decision Records (ADRs) by Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)**: Implemented via immutability and superseding. Extended by creating **Architecture Locks**: executable rules extracted from ADRs so agents respect them without reading the entire history.
3. **[Context Engineering Patterns (Karpathy LLM Wiki)](https://github.com/karpathy/LLM-Wiki)**: Materialized by drastically limiting the reading tokens allowed per role (Planner, Executor, Checker), preventing context saturation.

## ⚙️ Core Mechanisms

- **Anti-drift**: Use of stable *Decision keys* and short indexes (`ARCHITECTURE_LOCKS.md`) that the agent must respect when touching specific repository *paths*.
- **Anti-minimal-effort**: The agent cannot close a task without providing console evidence (pasted in the markdown) that a test failed (RED) for the expected reason before writing the functional implementation (GREEN).
- **Deterministic Handoffs**: The system is based on 3 roles (Planner, Executor, Checker) with unbreakable read/write/verify (R/W/V) permission matrices.

## 📖 Full Specification

Read the full framework document and its operational templates here:
👉 **[Framework Specification (framework-spec.md)](./framework-spec.md)**

## 🤝 Call to the Community (Request for Comments)

This framework is in the proposal phase, and I am looking for feedback from professionals operating with LLM agents in real-world environments. The main areas for debate are:

- **Token limits**: Are the stipulated reading thresholds realistic for 32B local models?
- **Blocking mechanisms**: Are there evasion vectors where an agent could bypass `verify_sprint.py` by altering the test output?
- **Edge cases**: Experiences trying to force aesthetic or UI refactoring under this strict TDD level.

Open an Issue to discuss the specification or submit a PR if you have improvements for the validation scripts.
