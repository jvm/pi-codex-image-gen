> [!IMPORTANT]
> This repository has moved to [`jvm/pi-mono`](https://github.com/jvm/pi-mono/tree/main/packages/pi-codex-image-gen). It is archived and no longer maintained here.
> Please file issues and pull requests in [`jvm/pi-mono`](https://github.com/jvm/pi-mono).

# pi-codex-image-gen

Image generation and editing for [Pi](https://github.com/badlogic/pi-mono) using the ChatGPT Images 2.0 model via the OpenAI Codex Responses backend.

## Install

```sh
pi install npm:pi-codex-image-gen
```

To uninstall:

```sh
pi remove npm:pi-codex-image-gen
```

## Quick usage

In a Pi session:

```
> Generate a pixel-art sword icon, 32×32, with a blue blade and gold hilt
```

The agent will invoke `codex_generate_image` with your prompt, stream the response from the Codex backend, and save the resulting image to disk. For edits of existing local images, the agent can invoke `codex_edit_image` with the source image path(s) and edit instructions. The `model` parameter controls the Codex routing model; image generation/editing is always performed by **gpt-image-2** on the backend.

## Authentication

Uses your existing **openai-codex** login — no `OPENAI_API_KEY` required. If you haven't logged in yet:

```
> /login
```

Select **ChatGPT Plus/Pro (Codex)** and complete the OAuth flow.

## Configuration

Create a JSON config file at one (or both) of these locations:

| Scope   | Path                                                    |
| ------- | ------------------------------------------------------- |
| Global  | `~/.pi/agent/extensions/codex-image-gen.json`           |
| Project | `<project-root>/.pi/extensions/codex-image-gen.json`    |

Project config overrides global config. Example:

```json
{
  "save": "global",
  "saveDir": "~/Pictures/generated",
  "model": "gpt-5.5"
}
```

### Config keys

| Key       | Type   | Default    | Description                              |
| --------- | ------ | ---------- | ---------------------------------------- |
| `save`    | string | `"global"` | Default save mode (see below).           |
| `saveDir` | string | —          | Directory used when `save=custom`.       |
| `model`   | string | `"gpt-5.5"`| Codex routing model. Image generation is always handled by gpt-image-2. |

### Environment variables

| Variable                     | Description                                      |
| ---------------------------- | ------------------------------------------------ |
| `PI_CODEX_IMAGE_SAVE_MODE`   | Overrides the `save` config key.                 |
| `PI_CODEX_IMAGE_SAVE_DIR`    | Overrides the `saveDir` config key (custom mode).|
| `PI_OFFLINE=1`               | Disables install/update telemetry.              |
| `PI_TELEMETRY=0`             | Disables install/update telemetry.              |

## Save modes

| Mode      | Behavior                                                         |
| --------- | ---------------------------------------------------------------- |
| `none`    | Image is returned inline but not written to disk.                |
| `project` | Saves to `<project>/.pi/generated-images/<session-id>/`.         |
| `global`  | Saves to `~/.pi/agent/generated-images/<session-id>/`.           |
| `custom`  | Saves to a user-specified directory (requires `saveDir` or env). |

## Tool parameters

### `codex_generate_image`

| Parameter      | Type   | Required | Description                                                        |
| -------------- | ------ | -------- | ------------------------------------------------------------------ |
| `prompt`       | string | ✅        | The image generation prompt.                                       |
| `model`        | string | —        | Override the Codex model. Defaults to config or `gpt-5.5`.         |
| `outputFormat` | string | —        | `png` (default), `jpeg`, or `webp`.                                |
| `save`         | string | —        | Override save mode for this call.                                  |
| `saveDir`      | string | —        | Directory when `save=custom`. Relative paths resolve under CWD.    |

### `codex_edit_image`

Edits existing local images through the same Codex Responses backend by sending source image files as `input_image` data URLs alongside the edit prompt. It does **not** use the OpenAI Images API and does **not** require `OPENAI_API_KEY`.

| Parameter      | Type              | Required | Description                                                     |
| -------------- | ----------------- | -------- | --------------------------------------------------------------- |
| `prompt`       | string            | ✅        | Natural-language edit instructions.                             |
| `image`        | string/string[]   | ✅        | Source image path(s). Relative paths resolve under CWD.          |
| `model`        | string            | —        | Override the Codex routing model. Defaults to config/`gpt-5.5`.  |
| `outputFormat` | string            | —        | `png` (default), `jpeg`, or `webp`.                              |
| `save`         | string            | —        | Override save mode for this call.                                |
| `saveDir`      | string            | —        | Directory when `save=custom`. Relative paths resolve under CWD.  |

Example:

```sh
pi --tools codex_edit_image -p 'Use codex_edit_image to edit ./duck.png: make the duck red, preserve the same silhouette, save global, return the path.'
```

## How it works

1. Resolves auth via Pi's `openai-codex` provider (ChatGPT session token).
2. Sends a Codex Responses API request to the routing model (default `gpt-5.5`) with the `image_generation` tool enabled.
3. For edits, includes source files as Responses `input_image` content items in the same request.
4. The backend invokes **gpt-image-2** to generate or edit the image.
5. Parses the SSE stream for `response.output_item.done` events containing the base64 image.
6. Saves the image to disk according to the active save mode.
7. Returns the image data inline plus metadata (model, format, path, revised prompt, usage).

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| "Missing openai-codex credentials" | Not logged in | Run `/login` and select **ChatGPT Plus/Pro (Codex)** |
| 401 / 403 response | Token expired | Re-run `/login` for openai-codex |
| 429 response | Rate limited | Wait and retry; the extension retries automatically with backoff |
| "Codex did not return an image" / "Codex did not return an edited image" | Backend refused the prompt or image input | Rephrase the prompt, verify the image path exists, and try again |
| "save=custom requires saveDir" | Missing config | Set `saveDir` in config or `PI_CODEX_IMAGE_SAVE_DIR` env var |

## License

Apache-2.0
