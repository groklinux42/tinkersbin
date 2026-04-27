# ADR-003: Three-Signal Duplicate Detection

**Status:** accepted
**Date:** 2026-04-27
**Author:** web-chat

## Context

When the user captures a part, the system needs to answer "is this something we've already seen?" The strongest signal is the manufacturer part number, but that's only readable when there are markings, the markings are legible, and the part is in fact a uniquely-identified component (not a generic resistor or unmarked SMD). Loose parts pulled from a bin may have no readable text at all. Vendor labels with truncated or missing P/N (the Amazon ESP32 case) leave only the listing title and image. A single-signal dedup will frequently fail in either direction — false negatives (creating duplicate part records) or false positives (collapsing distinct parts).

## Decision

Duplicate detection runs three signals in parallel and surfaces all matches to the user, who confirms or rejects:

1. **Manufacturer P/N exact match** — when extracted text contains a P/N that matches an existing part record. Highest confidence when present.
2. **Perceptual hash (pHash) image similarity** — every captured image gets a pHash; new captures are compared against stored part_images for near-matches. Catches the unmarked-loose-part case where text-based matching is impossible.
3. **Text similarity on description/title** — fuzzy match (e.g. trigram or similar) against existing part descriptions and stored vendor titles. Backup signal for cases where the P/N reads incorrectly but the rest of the text is recognizable.

The UI shows all three results side-by-side. The user picks one (it's the same part, log a new lot), rejects all (it's a new part, create a new record), or partially confirms (these candidates are close but wrong, here's a new record but link them as related/alternate).

## Consequences

- Schema must store pHash on every part_image row, with an index suitable for similarity search (the specific approach is an implementation choice — exact-match index on pHash buckets, BK-tree, or similar).
- Schema must support stored vendor titles/descriptions as searchable text, separate from user-entered fields, so we don't conflate scraped data with curated data.
- The UI cost is real: one capture = up to three lists of candidates plus the "new part" path. This must not feel onerous. Default flow when a single high-confidence P/N match exists should be one tap to confirm.
- False positives from any single signal are tolerable because the user is the final filter. The cost of showing a wrong candidate is trivial; the cost of silently merging two distinct parts is high.
- Adding a fourth or fifth signal later (vendor SKU exact match is an obvious one) is additive — the surface stays the same.
