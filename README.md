# aipg-frontends

Drop-in configs to point popular chat frontends at the
[AI Power Grid](https://api.aipowergrid.io/v1) OpenAI-compatible API — so you can
use the grid's models in a full-featured UI in a couple of minutes.

- **[LibreChat](#librechat)** — ready `librechat.yaml` custom endpoint.
- **[Open WebUI](#open-webui)** — step-by-step connection (+ optional image generation).

> Unofficial, community-built. Not affiliated with AI Power Grid.
> Base URL: `https://api.aipowergrid.io/v1` · get a key from the AIPG dashboard.

## What you get

The grid speaks the OpenAI API, so both UIs can **auto-fetch the model list** from
`/v1/models`. Current text/vision models (the live list may vary):

| Model | Notes |
|---|---|
| `deepseek-v4-flash-nvfp4` | Fast, **non-reasoning**, supports **vision** (image input). Good default. |
| `gpt-oss-120b` | Large, **reasoning** model — see the max-tokens note below. |
| `qwen3:8b` (and other `qwen3:*`) | **Reasoning** models — see the max-tokens note below. |

Image generation is also available (`z-image-turbo`, `FLUX.2 Klein 4B FP8`, …) —
see the Open WebUI image section.

## ⚠ Read first — two gotchas

1. **Reasoning models can return empty messages at low max-output-tokens.**
   `qwen3:*` and `gpt-oss-120b` spend the token budget on hidden reasoning; if the
   UI caps output tokens too low you get a blank reply (`finish_reason: length`).
   **Fix:** set a generous *max output tokens* (≥ ~1024, more for long answers), or
   just use `deepseek-v4-flash-nvfp4` as your default — it doesn't do hidden
   reasoning. (Background: AIPowerGrid/grid-skill#22.)
2. **No embeddings endpoint.** `/v1/embeddings` returns 404, so don't point a
   frontend's **RAG/embedding** setting at the grid — keep embeddings on the UI's
   built-in local model or another provider. The grid handles chat, vision and
   image generation.

---

## LibreChat

1. Copy [`librechat/librechat.yaml`](librechat/librechat.yaml) next to your
   LibreChat `docker-compose.yml` (or merge the `endpoints.custom` entry into your
   existing file).
2. Put your grid key in LibreChat's `.env`:
   ```env
   AIPG_API_KEY=grid_your_key_here
   ```
3. Restart LibreChat. "AI Power Grid" appears as an endpoint; models are fetched
   automatically.

The provided config sets `deepseek-v4-flash-nvfp4` as the title/default model (no
reasoning surprises). Users can raise *Max Output Tokens* per conversation for the
reasoning models.

## Open WebUI

1. **Admin Panel → Settings → Connections → OpenAI API → `+`**
   - **API Base URL:** `https://api.aipowergrid.io/v1`
   - **API Key:** your `grid_…` key
2. Save. Open WebUI fetches the model list; pick a model and chat.
3. **Recommended:** Admin → Settings → Models → set *Max Tokens (num_predict)* to
   `1024`+ for `qwen3:*` / `gpt-oss-120b`, or default users to
   `deepseek-v4-flash-nvfp4`.

### Optional: image generation in Open WebUI

**Admin Panel → Settings → Images**
- **Image Generation Engine:** `OpenAI`
- **API Base URL:** `https://api.aipowergrid.io/v1`
- **API Key:** your `grid_…` key
- **Model:** `z-image-turbo` (a model is **required** — the grid rejects `auto`
  for images; other options: `FLUX.2 Klein 4B FP8`, `Krea 2 Turbo`,
  `MOSS Image SFT BF16`)
- **Image Size:** keep within **512×512 – 1536×1536** (the grid validates this).

### Note on RAG / embeddings

Leave Open WebUI's embedding model on its **default (local)** setting. The grid has
no `/v1/embeddings`, so selecting it there would break document RAG.

## License

MIT — see [LICENSE](LICENSE).
