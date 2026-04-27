# Tinkersbin

## Purpose

A personal electronics parts inventory system for hobbyists. The user has accumulated boxes of components from years of online ordering and wants to *know* what's on hand rather than guess. Tinkersbin captures parts via phone camera, identifies them through a vision-language model, enriches them from the original vendor's product page, and tracks them across multiple purchase lots so duplicate-buying can be avoided. It also accepts a Bill of Materials and produces a "what do I need to buy" shopping list against current inventory.

## Scope

- Capture parts via phone camera (live or from gallery), submit to a vision pipeline for identification
- Identify generic class, manufacturer, manufacturer P/N, package, and any visible markings
- Enrich identified parts by fetching the original vendor's product page (Adafruit, Amazon, DigiKey, Mouser, etc.)
- Track parts as lots — each purchase is a lot, parts can have many lots, on-hand quantity is the sum of remaining lot quantities
- Detect duplicate parts at capture time using three signals: manufacturer P/N, perceptual hash of image, text similarity on description
- Accept a BoM (CSV from KiCad/EAGLE, or hand-typed text) and produce a shopping list showing what's on hand vs. what's needed
- Single-user; runs on the user's own infrastructure

## Non-Goals

- Multi-user, teams, or any form of collaboration
- Authentication beyond what Tailscale provides at the network layer
- Native mobile app — phone access is via mobile browser
- ERP/MRP features: production runs, work orders, labor tracking, cost accounting
- PCB design, schematic capture, or CAD integration
- Inventory categories beyond electronics (no chemicals, no retail, no general-purpose stock)
- Counterfeit detection or component lifecycle/EOL tracking
- Public hosting as a service for others
- E-commerce integration beyond fetching product pages (no order placement, no API auth to retailers)

## Tech Stack

- **Webapp** (Hetzner): PHP, MariaDB. Single-line PDO queries, no prepared statements, functionality over convention. Mobile-first responsive UI.
- **Vision/inference service** (HOME-DEV): Python, llama.cpp running Qwen3-VL-9B on the RTX 4070Ti. PaddleOCR available as secondary text extractor. HTTP service exposed only on Tailscale.
- **Network**: Tailscale connects Hetzner to HOME-DEV. Phone reaches Hetzner via public web or Tailscale.
- **Domain**: tinkersbin.com

## Constraints

- Self-hosted, no SaaS dependencies for core function
- The vision GPU stays at home; the always-on web frontend is in the cloud; nothing breaks when the home connection blips except live capture
- Vendor scraping is best-effort and tolerant of failure — if Amazon changes its DOM the system still works, it just doesn't auto-enrich that part
- Code style: Grok's preferences govern. Single-line PDO, no ORM, no framework ceremony. Python service is a small Flask/FastAPI-style HTTP wrapper around llama.cpp inference, not a microservice menagerie.

## Repository Structure

```
tinkersbin/
├── .context/         # Code Bridge protocol files
├── webapp/           # Hetzner-side PHP/MariaDB app (planned)
├── eye/              # HOME-DEV-side Python inference service (planned)
├── docs/             # Design notes, retailer adapter notes (planned)
├── CLAUDE.md         # Protocol instructions
└── README.md
```

## Key User Context

The user is picking up the SpeechWrite ZVI project and needs to know what audio/microcontroller parts are already on hand before ordering more. The PCM5102 DAC and ESP32-S3 boards already test-photographed are part of that build. Tinkersbin is being built in service of that build (and many like it after).
