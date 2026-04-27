# ADR-001: Two-Host Topology with Tailscale

**Status:** accepted
**Date:** 2026-04-27
**Author:** web-chat

## Context

Tinkersbin needs an always-on web endpoint reachable from a phone, plus GPU-backed vision inference for part identification. The user owns a Hetzner cloud box and a home workstation (HOME-DEV) with an RTX 4070Ti running Qwen3-VL-9B locally. Putting everything on Hetzner forfeits the GPU; putting everything on HOME-DEV ties availability to a residential connection and creates DDNS/firewall complexity.

## Decision

Tinkersbin runs as a two-host system. Hetzner is the always-on web frontend (PHP/MariaDB, public on tinkersbin.com). HOME-DEV runs the inference service ("the eye") on a Tailscale-only address. The webapp calls the eye over Tailscale; the eye is never exposed to the public internet. The phone reaches the webapp either over the public internet or over Tailscale, the user's choice.

## Consequences

- The web frontend stays responsive when HOME-DEV is offline; only live capture/identification breaks. Inventory browsing, BoM matching, and history queries continue to work because they only need the database, which lives on Hetzner.
- The GPU never needs public exposure — no auth code to write at the inference layer beyond what Tailscale provides at the network layer.
- Image storage is a deployment-time decision (Hetzner vs. HOME-DEV vs. proxied). See ADR for image storage when written.
- Bandwidth cost: every captured photo crosses Tailscale at least once. Acceptable for the scale of personal use.
- If HOME-DEV is replaced or augmented (e.g. moving inference to the Orin Nano Super), the eye contract is the only thing that has to be re-implemented — the webapp doesn't move.
