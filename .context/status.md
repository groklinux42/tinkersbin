# Project Status

**Last Updated:** 2026-04-27 by web-chat

## Current Phase

Foundation. Architecture captured in ADRs, repo scaffolded, first build task ready: establish the database schema.

## Components

| Component | Status | Owner | Notes |
|-----------|--------|-------|-------|
| Architecture / ADRs | Done | web-chat | ADR-001 through ADR-006 in place |
| Repo scaffold | Done | web-chat | Code Bridge protocol set up |
| Database schema | Planned | code | Task 001 — first handoff |
| Webapp skeleton | Planned | code | After schema lands |
| Eye service skeleton | Planned | code | Parallel-able with webapp once schema is set |
| Capture endpoint | Planned | code | Depends on schema + webapp + eye contract |
| Adafruit adapter | Planned | code | First retailer adapter (most deterministic) |
| Amazon adapter | Planned | code | Second retailer adapter |
| BoM matching | Planned | code | Late-stage, after capture works end-to-end |
| Domain & deployment | Planned | human | tinkersbin.com registration + Hetzner setup |

## Active Questions

- Image storage location: Hetzner disk vs. HOME-DEV disk vs. Hetzner with HOME-DEV as cold backup. Will be addressed in a future ADR before the capture endpoint is built. Not blocking schema work.
- VLM size: Qwen3-VL-9B is the target, but the user wants to test it first against real captures. Not blocking schema work.
- Phone access mode: Tailscale-only vs. public + auth. Not blocking schema work; affects deployment and webapp auth scaffolding only.

## Known Issues

- None
