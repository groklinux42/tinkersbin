# ADR-006: Tech Stack and Code Style

**Status:** accepted
**Date:** 2026-04-27
**Author:** web-chat

## Context

The user has 30 years of programming experience and strong opinions about tooling. He prefers functionality over convention, single-line PDO queries without prepared statements, no ORM ceremony, and minimal framework overhead. He has explicitly rejected Python/Django and Docker as overused and "not usually appropriate" for this kind of project. He runs PHP/MariaDB at work daily and is fluent in Python for inference-side work.

Multiple existing inventory tools (InvenTree, Binner, PartKeepr) were considered and rejected — InvenTree for the Python/Django/Docker stack, Binner for the .NET dependency, PartKeepr for being archived and stuck on Symfony 2. The decision to build from scratch is documented in the conversation history that led to this project.

## Decision

**Webapp (Hetzner side):**
- PHP (recent stable version), no framework
- MariaDB
- PDO for database access, single-line queries, no prepared statements
- Mobile-first responsive HTML/CSS, vanilla JS where needed
- No build step, no asset pipeline, no transpiler — files are deployed as-is

**Eye service (HOME-DEV side):**
- Python
- llama.cpp running Qwen3-VL-9B with full GPU offload (`-ngl 99`)
- HTTP server (Flask, FastAPI, or comparable thin framework — implementer's choice within reason)
- PaddleOCR available as a secondary text extractor

**Networking:**
- Tailscale for Hetzner ↔ HOME-DEV
- Public web on tinkersbin.com via Hetzner; phone access via public web or Tailscale (user's choice per session)

**Repository style:**
- One repo, two top-level directories: `webapp/` and `eye/`
- Each is independently deployable
- No monorepo tooling; they are simply colocated source trees

## Consequences

- Code review standards align with the user's style. Pull requests that introduce ORM layers, dependency injection containers, prepared statements, or framework abstractions for their own sake will be rejected.
- The Python service is intentionally small — one HTTP wrapper, model invocation, response shaping. It is not a microservice menagerie.
- New contributors (including future Claude Code sessions) must read this ADR before making stylistic choices. Reference implementations in the repo are the source of truth once they exist.
- "Functionality over convention" does not mean "no structure" — it means structure earns its place by solving an actual problem, not by being expected.
