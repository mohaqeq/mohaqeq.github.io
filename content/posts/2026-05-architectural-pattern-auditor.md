---
title: "Discovering a codebase's architecture instead of declaring it"
date: 2026-05-24
tags: [software-architecture, tools, ai-agents, static-analysis]
summary: "An agent skill that finds the architectural patterns and anti-patterns a repository actually has, across 13 languages, and grades each one with file:line evidence."
---

Architecture tools usually start from a rule you have already written down. ArchUnit, import-linter and
dependency-cruiser are excellent at *enforcing* "the domain must not import the web layer". But when you open an
unfamiliar codebase, the first question is different: *what architecture does this actually have?* Is it layered,
hexagonal, a modular monolith or a big ball of mud? Where does it break its own pattern?

I built [architectural-pattern-auditor](https://github.com/mohaqeq/architectural-pattern-auditor) to answer that. It
is a Claude Code [agent skill](https://agentskills.io/specification), but every step is a plain script you can run
without an AI.

## What it produces

Two files: `architecture-audit.md` for humans and `architecture-audit.json` for tools. Findings are grouped in three
levels:

- **Macro, the system shape:** monolith, modular monolith, microservices, event-driven, hexagonal, clean/onion,
  layered, CQRS, event sourcing, serverless, plugin architecture and more.
- **Meso, components:** MVC/MVVM/MVP, repository, unit of work, service layer, DDD aggregate, saga, outbox, circuit
  breaker, strangler fig, API gateway and others.
- **Micro, idioms:** the classic design patterns (factory, builder, strategy, observer, decorator, adapter…) and
  dependency injection.

It also looks for anti-patterns: big ball of mud, distributed monolith, anemic domain model, god object, circular
dependencies, layering violations, leaky abstractions, shotgun surgery and feature envy.

## How it works

A pipeline of five scripts:

```bash
python scripts/scan_repo.py          --root "$ROOT" --mode quick --out "$OUT"
python scripts/build_import_graph.py --root "$ROOT" --out "$OUT/graph.json"
python scripts/detect_layers.py      --graph "$OUT/graph.json" --out "$OUT/layers.json"
python scripts/detect_patterns.py    --root "$ROOT" --inventory "$OUT/inventory.json" \
                                     --graph "$OUT/graph.json" --layers "$OUT/layers.json" --out "$OUT/findings.json"
python scripts/render_report.py      --findings "$OUT/findings.json" --template templates/audit-report.md.j2 ...
```

It inventories the repository, builds an import graph across **13 languages** (Python, TypeScript/JavaScript, Java,
Kotlin, Scala, Groovy, Go, Rust, C#, Ruby, PHP, Swift, Dart), infers layers from the graph, and then runs structural
checks per pattern. `--mode quick` uses folder and name heuristics for a first pass; `--mode deep` runs the full
per-file checks.

## Conservative by design

The easiest way to make a tool like this useless is to let it guess. So it has three rules:

1. **No pattern without two independent structural signals.** A folder called `domain/` alone proves nothing.
2. **Every "Incorrect" verdict carries file:line evidence**, so a human can check it in seconds.
3. **Degradation is visible.** It works offline and degrades gracefully: without ripgrep it uses a pure-Python grep;
   without networkx, a bounded depth-first search for cycles; without jinja2, a small built-in template engine. Every
   fallback is recorded in the findings and shown in the report's appendix, so you know how much to trust a run.

## Why it's a skill

Inside Claude Code you can just ask: *"audit the architecture of this repo"*, *"is this hexagonal?"*, *"find
architectural anti-patterns"*. The agent runs the scripts and reads the evidence. The deterministic scripts do the
detecting, and the model explains the results. Per-language cheatsheets of idiomatic signals and a pattern catalogue
ship with the skill as references.

## Where it fits

This is the *discovery* half of architecture work. Enforcement is the other half, and I explored it later in a much
larger experiment: [a checked notation for rules and decisions](/posts/2026-11-agents-already-read-the-code/). Discover
first, then write down the rules you want to keep, then let a checker hold the line.
