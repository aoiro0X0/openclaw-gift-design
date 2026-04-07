# openclaw-gift-design

OpenClaw skill for Douyin Live gift design — covers the full workflow from ops document intake to image delivery.

## What it does

- **Mode A — Ops doc intake**: reads a Feishu ops document, extracts every gift item and price, matches each to the Douyin gift 价效梯度 table, outputs a compliance table, and auto-creates a design work document in Feishu.
- **Mode B — Design workflow**: analyzes designer intent (multi-reference composition/color roles, price-tier check), routes to the right model (Seedream for creative txt2img, Gemini for editing/inpaint/background-replace), runs image generation, and returns results via OpenClaw media fields.

## Install

```text
Use $skill-installer to install this skill from https://github.com/aoiro0X0/openclaw-gift-design/tree/main/openclaw-gift-design
```

## Environment variables

| Var | Purpose |
|-----|---------|
| `ZENMUX_API_KEY` / `GEMINI_API_KEY` | Image generation API key (one required) |
| `ZENMUX_BASE_URL` | Override Vertex AI base URL |
| `OPENCLAW_BANANA_MODEL` / `ZENMUX_IMAGE_MODEL` | Force a specific image model |
| `BANANA_MODEL_REGISTRY` | JSON array to override the model registry |