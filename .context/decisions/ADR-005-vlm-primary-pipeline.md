# ADR-005: VLM-Primary Vision Pipeline

**Status:** accepted
**Date:** 2026-04-27
**Author:** web-chat

## Context

PaddleOCR is an OCR model — it reads text. It does not classify. The original conversation considered PaddleOCR as the primary identifier, but identification of an electronic part is a recognition task, not a text-extraction task. Modern open-weight vision-language models (Qwen3-VL-9B in particular, which the user is targeting) handle the full task in one shot: classify the generic component, read any visible markings, infer the package format, comment on visible features (polarity stripes, antenna traces, header layouts), and produce structured output.

The two test captures (Adafruit PCM5102 in a bag, Amazon ESP32-S3 3-pack in a mylar bag) both contained vendor labels with the most useful information. A VLM can identify both labels and parts in a single pass. OCR alone would handle the labels but miss the boards; classification alone would handle the boards but miss the labels.

## Decision

The eye service uses Qwen3-VL-9B as the primary identifier, running on llama.cpp with full GPU offload on the RTX 4070Ti. It returns structured JSON with: generic class, manufacturer, manufacturer P/N (if visible), vendor SKU/identifier (if visible), vendor URL (if visible), package, visible markings/text array, value estimates where applicable, perceptual hash, and confidence notes including any ambiguities.

PaddleOCR is retained as a secondary tool, called when the VLM reports low confidence on text extraction or when the captured image is text-dominant (e.g. a closeup of a reel label). The eye decides internally whether to invoke it; the webapp doesn't see this distinction.

The eye is an HTTP service with a single primary endpoint: POST an image, get JSON back. The contract is what the webapp depends on — the model, the prompting strategy, the secondary tools, and the inference framework can all change without the webapp noticing.

## Consequences

- The webapp is decoupled from inference details. Swapping Qwen3-VL-9B for something larger (or smaller, on the Orin) or switching from llama.cpp to vLLM is purely an eye-side change.
- The eye's output schema needs to be stable enough that the webapp can rely on it. A versioned response format is appropriate from day one (e.g. `"schema_version": 1`).
- Confidence reporting matters. The webapp should be able to distinguish "I'm sure this is an Adafruit PCM5102" from "this is probably an ESP32 board, can't tell S2 vs S3." Low-confidence results should surface to the user as ambiguity rather than be silently committed.
- Inference latency is real (seconds per image, not milliseconds). The webapp should treat capture as asynchronous: image accepted, identification proceeds in the background, UI updates when the eye replies.
- The eye's prompt to the VLM is itself an artifact worth versioning. Prompt changes are tracked alongside code changes because they affect output shape.
