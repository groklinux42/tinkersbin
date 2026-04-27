# ADR-002: Lots-Not-Counts Inventory Model

**Status:** accepted
**Date:** 2026-04-27
**Author:** web-chat

## Context

Hobbyists buy the same part multiple times — sometimes because they ran out, more often because they were unsure whether they had it and bought another to be safe. A naive "parts have a quantity" model collapses these purchases into a single number and discards the history. This makes "have I bought this before?" impossible to answer at capture time and forces mental reconciliation when restocking.

## Decision

Parts are tracked as a parent record with one or more child lot records. A lot represents a single purchase event: an order reference, an initial quantity, a current remaining quantity, an optional storage location, and any per-lot notes. On-hand quantity for a part is the sum of `qty_current` across that part's lots. When the user captures a part that already exists, the system creates a new lot under the existing part rather than incrementing a counter.

## Consequences

- "I bought this part six months ago and don't remember" is answerable directly: capture detects the existing part, shows existing lots with their dates, lets the user add a new lot or just confirm the existing one.
- Lot-level traceability comes free: bad batch from a vendor, counterfeit run, or mis-described product can be tracked to the specific lot that was affected.
- BoM consumption can decrement specific lots (FIFO, LIFO, or user-chosen), preserving history of which lot fed which build. Whether the system implements automatic decrementing on build is a separate decision.
- Slight UX cost: "how many do I have" requires a sum, not a column read. Trivial in SQL, negligible in practice.
- Per-lot fields like `storage_location` allow physically distributing a single part type across multiple bins (e.g. workshop drawer + travel kit) without inventing a separate concept.
