---
name: see-3d-model
description: Look at a GLB model from several angles and read its geometry stats, so you can judge a generated or downloaded 3D asset. Use after generating a model, or whenever you need to verify what a GLB URL actually contains.
---

# See a 3D model

A `.glb` is opaque to a language model. This gives you frames you can actually look at, plus geometry facts and a plain reading of them. Free, no key, no account.

## MCP (preferred in this plugin)

The plugin's `.mcp.json` registers the keyless server `https://three.ws/api/mcp-studio`. Its `look_at_model` tool returns the rendered angles as MCP image content blocks, so you see them directly in the conversation. Pass the public GLB URL.

## Library

```bash
npm install @three-ws/see
```

```js
import { see, toMessageContent } from '@three-ws/see';

const look = await see('https://example.com/robot.glb', { views: ['front', 'back'], size: 768 });
look.views;     // [{ view, theta, phi, imageUrl }, ...]
look.notes;     // plain-language reading, e.g. triangle budget
look.stats;     // { vertices, triangles, materials, textures, animations, ... }
look.viewerUrl; // spin it in a browser
look.arUrl;     // place it in a room from a phone
look.missingViews; // angles that failed to render, if any
const content = await toMessageContent(look); // labelled text + image blocks for a multimodal chat
```

`toMessageContent(look, { fetchImages: true })` inlines base64 for APIs that will not fetch URLs.

## Views and options

Views: `three-quarter`, `front`, `side`, `back`, `top`, `bottom` (default: three-quarter, front, side, back; up to 6 per call; unknown names are ignored). Options: `views`, `size` (128 to 1024, default 512), `stats` (default true), `baseUrl` (default `https://three.ws`), `timeoutMs` (default 120000), `fetch`.

What each view catches: `back` is where single-view generators leave work unfinished; `side` shows flattened geometry; `bottom` shows missing caps and inverted normals; `top` shows hollowness.

## Judge loop

Generate, then `see` the result, then critique against the intent: is the subject complete and recognisable, is the far side finished, is anything melted or fused? Regenerate or fix with the specific problem named.

## Errors

`SeeError` with `code`: `invalid_url`, `render_failed`, `rate_limited` (wait `err.retryAfter` seconds), `timeout`, `unreachable`. A partial turntable is not an error; use the frames returned and note `missingViews`. REST underneath: `POST /api/3d/look`.
