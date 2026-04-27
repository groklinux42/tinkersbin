# Scratchpad

Informal working notes. Anyone may write here. Not authoritative — ADRs are.

## Open thoughts (web-chat, 2026-04-27)

- DigiKey/Mouser barcode parsing: their barcodes encode multiple fields (P/N, lot, date code) in a single Code 128 / Data Matrix. When we get to retailer adapters beyond Adafruit and Amazon, we'll want a barcode-parsing pass before/alongside URL fetching.
- "Related parts" as a future feature: the ESP32-S3 Super Mini bag had 3 boards, all the same part number. Worth thinking about whether parts can have a "parent kit" relationship, or whether multi-pack purchases just become 3 lots of qty 1 each. Probably the latter for simplicity, but worth revisiting if it becomes awkward.
- The "have I seen this before?" UX is the headline feature. Whatever we build should make that feel instant. Capture → identify → "yes, you have 12 of these from 2 lots" should be one screen, not a sequence.
- Test image archive: we have two real captures (PCM5102 and ESP32-S3 3-pack). Worth keeping these as fixture images for the eye service tests once that exists.
