# ADR-004: Vendor URL as Primary Enrichment Path

**Status:** accepted
**Date:** 2026-04-27
**Author:** web-chat

## Context

Test captures showed two distinct enrichment situations. The Adafruit bag carries a printed product URL (`adafru.it/6250`) and a SKU; fetching that URL gives a canonical product page with title, description, image gallery, and links to datasheets — everything we want to enrich a part record. The Amazon bag carries an ASIN (`X0048PAJWN`) and a truncated title; the ASIN deterministically maps to `amazon.com/dp/{ASIN}`, which provides similar enrichment if we can fetch and parse it. DigiKey, Mouser, LCSC, Sparkfun, and others each have their own label format and URL convention.

Trying to identify parts purely from visual signal — even with a strong VLM — leaves too much on the table when the bag itself contains a deterministic vendor reference.

## Decision

When a vendor URL or vendor identifier (SKU, ASIN, DigiKey P/N) is visible at capture time, the system fetches the corresponding product page server-side and uses that as the primary enrichment source. Each retailer is implemented as a small adapter with a uniform contract: given a vendor identifier or URL, return a normalized product record (title, description, images, retailer P/N, mfr P/N if listed, datasheet links if found).

Adapters are best-effort: if a retailer changes their DOM, that adapter degrades to "we got the title and that's it" rather than crashing the capture. The webapp does not block on enrichment — capture succeeds with whatever the eye returned, enrichment fills in afterward and the UI updates when it's available.

The user can also paste a vendor URL manually when re-photographing an old bag whose label is illegible, or when adding a part that came in plain packaging.

## Consequences

- Retailer adapters are the right abstraction boundary. Each one is small (a fetch + a few CSS/XPath selectors + a normalizer) and isolated. Adding a retailer is additive; one retailer breaking does not affect others.
- The vision pipeline's job becomes "find a vendor identifier if one exists, fall back to part identification otherwise" rather than "identify everything." This shifts work from inference to scraping, which is where work belongs when a deterministic answer is available.
- Adafruit's product URL format (`adafru.it/{NNNN}`) and Amazon's ASIN-to-URL mapping are special cases worth handling first because they're trivially deterministic. DigiKey/Mouser barcode parsing is more involved (they encode multiple fields in a single barcode) and can come later.
- The webapp must tolerate failed enrichments gracefully. A part with no vendor URL and only visual identification is still a valid part record.
