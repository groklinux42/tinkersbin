# Handoff: Task 001 — Database Schema

**Date:** 2026-04-27
**Branch:** main
**From:** web-chat
**To:** code

## Task

Establish the MariaDB schema that all subsequent webapp work will build on. This is a schema-only task — no application code, no seed data.

## READ FIRST

In order:
1. `.context/project.md` — project scope and tech stack
2. `.context/decisions/ADR-002-lots-not-counts.md` — the inventory model the schema must reflect
3. `.context/decisions/ADR-003-three-signal-dedup.md` — what dedup signals must be storable and queryable
4. `.context/decisions/ADR-004-vendor-url-enrichment.md` — what vendor/order data needs persisting
5. `.context/decisions/ADR-006-tech-stack-and-style.md` — code style constraints
6. `.context/handoff-to-code.md` — this file

## Reference Implementation

None. This is the first build task in the project.

## Intent

A hobbyist captures a part, the system identifies it, links it to the order it came from, tracks how many remain, and answers "have I seen this before?" three different ways. The schema is the bedrock that makes all of that possible. Get the entities and relationships right and everything else falls out naturally; get them wrong and we'll be migrating for months.

## Entities and Relationships

The schema needs to represent (at minimum):

**Parts** — the canonical identity of a kind of component. A part is a thing like "Adafruit PCM5102 I2S DAC" or "10kΩ 0805 1% resistor (generic)." User-facing fields include manufacturer, manufacturer P/N, generic class, package, value, description, and free-form notes. A part has zero or more lots, zero or more images, and may carry a primary vendor reference if it's a single-vendor product.

**Lots** — a single purchase event of a part. A lot has an initial quantity, a current remaining quantity, an optional storage location, an optional reference to an order, and per-lot notes (e.g. "left bench drawer," "found short by 3 in this batch"). On-hand quantity for a part is the sum of `qty_current` across its lots. Lots are the unit of provenance.

**Orders** — a purchase from a retailer, separate from lots so multiple lots from the same order can be linked. An order has a retailer name, the original product URL, the date ordered, and stored scraped data from the vendor product page (title, body, image references). Scraped data is kept verbatim for forensic value — if the retailer page later changes or disappears, we still have what was on it the day we bought.

**Part images** — every captured photo is an image record linked to a part. Each image carries a perceptual hash, a source classification (capture, vendor page, datasheet, user-uploaded), and the path to the actual file. The pHash column must be queryable for similarity, which is the core of ADR-003 signal #2.

**BoMs and BoM lines** — a parsed Bill of Materials, each line tied (when matched) to a specific part. A BoM has a name and creation date; lines have raw input text, parsed mfr P/N, parsed value, quantity needed, and a nullable foreign key to the matched part. Unmatched lines stay unmatched until the user resolves them.

## Expected Outcomes (Acceptance Criteria)

The delivered schema should make all of the following queries straightforward, given just the schema and SQL:

- [ ] Given a manufacturer P/N, find the matching part record (signal #1 from ADR-003).
- [ ] Given a pHash, find part images with similar pHashes (signal #2 from ADR-003). The specific similarity strategy is your call — exact-match against bucketed pHash, BK-tree, hamming-distance scan, or other. Document which approach you chose and why.
- [ ] Given a description fragment, find parts with similar descriptions (signal #3 from ADR-003). Approach is your call.
- [ ] Given a part, list all its lots with their current remaining quantities and the orders they came from.
- [ ] Given a part, return the total on-hand quantity (sum of `qty_current` across lots).
- [ ] Given an order, list all lots that came from it.
- [ ] Given a BoM, list lines that have a matched part and lines that don't, with quantity-needed for each.
- [ ] Given a BoM and current inventory, derive a shopping list: for each line, on-hand vs. needed vs. shortfall.

The schema should be delivered as one or more numbered SQL files in `webapp/schema/` (e.g. `001-initial.sql`). Future migrations will follow the same numbering. No migrations framework — these are .sql files applied in order, per ADR-006.

Include reasonable indexes for the queries above. Don't over-index speculatively, but do index what the queries actually need.

Document your decisions inline in the SQL as comments, especially:
- Why you picked the pHash similarity approach you picked.
- Why you picked the text-similarity approach you picked.
- Any non-obvious column choices (e.g. why `value` is text vs. numeric, why `qty_current` is what it is).

## Out of Scope

- Application code (no PHP yet)
- Seed/sample data
- A migrations framework or runner script
- The webapp itself
- The eye service
- User accounts or authentication tables (single-user system; revisit if that ever changes)

## Style Notes

- Per ADR-006, no ORM-shaped naming conventions. Tables and columns should read naturally to a SQL person, not echo PHP class names.
- Single-line PDO queries are the consumption pattern. Schema should be friendly to that — avoid table designs that require complex multi-statement transactions for the common operations above.
- MariaDB-specific features are fair game (we're not pretending to be cross-database).

## Relevant ADRs

- ADR-002 (lots-not-counts): inventory model
- ADR-003 (three-signal dedup): query support requirements
- ADR-004 (vendor URL enrichment): orders table data shape
- ADR-006 (tech stack and style): SQL style and approach

## Questions for human (via web-chat)

None at this stage. If something here is ambiguous, surface it in `handoff-to-chat.md` rather than guessing — better to clarify than to commit a schema we'll regret.

## Claude Code Prompt

```
Read CLAUDE.md, then .context/project.md, then .context/decisions/ADR-002-lots-not-counts.md, then .context/decisions/ADR-003-three-signal-dedup.md, then .context/decisions/ADR-004-vendor-url-enrichment.md, then .context/decisions/ADR-006-tech-stack-and-style.md, then .context/handoff-to-code.md. Execute the task. Follow the context bridge protocol in CLAUDE.md.
```
