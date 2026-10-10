# @three-ws/see

[![npm](https://img.shields.io/npm/v/%40three-ws%2Fsee?color=cb3837&logo=npm)](https://www.npmjs.com/package/@three-ws/see)
[![license](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
[![node](https://img.shields.io/node/v/%40three-ws%2Fsee?color=339933&logo=node.js)](package.json)
[![zero deps](https://img.shields.io/badge/runtime%20deps-0-brightgreen)](package.json)
[![docs](https://img.shields.io/badge/docs-site-7c3aed)](https://nirholas.github.io/see/)
[![part of three.ws](https://img.shields.io/badge/part%20of-three.ws-111)](https://three.ws)

**Give an AI agent eyes for 3D.**

Every text-to-3D API on earth answers with a URL to a binary file. A human clicks it. An agent cannot: a `.glb` is opaque to a language model, so the agent that just generated an asset has no way to check its own work. That is why agentic 3D has been stuck at one shot and a shrug.

This package closes the loop in one call. Hand it a GLB URL and get back frames of the model rendered from several angles, ready to put in front of any multimodal model, plus the geometry facts and a plain reading of what they mean.

```
npm install @three-ws/see
```

No key. No account. The service is free.

## Look at a model

```js
import { see } from '@three-ws/see';

const look = await see('https://example.com/robot.glb');

look.views;
// [ { view: 'three-quarter', theta: 35, phi: 78, imageUrl: 'https://…/a.png' },
//   { view: 'front', … }, { view: 'side', … }, { view: 'back', … } ]

look.notes;
// [ '12,400 triangles, a normal real-time budget for a hero prop or character.' ]

look.stats;      // { vertices, triangles, materials, textures, animations, … }
look.viewerUrl;  // spin it in a browser
look.arUrl;      // place it in a real room from a phone
```

## Close the loop: generate, look, judge, fix

The reason this exists. An agent that can see its own output stops guessing.

```js
import Anthropic from '@anthropic-ai/sdk';
import { see, toMessageContent } from '@three-ws/see';

const anthropic = new Anthropic();

async function critique(glbUrl, intent) {
  const look = await see(glbUrl);
  const message = await anthropic.messages.create({
    model: 'claude-sonnet-5',
    max_tokens: 512,
    messages: [{
      role: 'user',
      content: [
        { type: 'text', text: `This model was meant to be: ${intent}. Judge it. Is the subject complete and recognisable, is the far side finished, is anything melted or fused? If it needs work, say exactly what to change.` },
        ...(await toMessageContent(look)),
      ],
    }],
  });
  return message.content[0].text;
}
```

`toMessageContent` shapes a look into multimodal chat content: one labelled text block per angle followed by the image itself. It passes image URLs by default; `{ fetchImages: true }` inlines base64 for APIs that will not fetch a URL themselves.

## Pick your angles

```js
await see(glbUrl, { views: ['back', 'top'], size: 768 });
```

| View | What it catches |
| --- | --- |
| `three-quarter` | Form and depth, the way a product shot reads (default first) |
| `front` | Facing, symmetry, the subject's identity |
| `side` | Profile, depth collapse, flattened geometry |
| `back` | The half a single-view generator most often leaves unfinished |
| `top` | Footprint, layout, whether it is hollow |
| `bottom` | Missing caps, inverted normals, floating geometry |

Up to six views per call, `size` from 128 to 1024 pixels (default 512). Unknown angle names are ignored rather than failing the call, so a guess still returns something useful.

## Errors are typed

```js
import { see, SeeError } from '@three-ws/see';

try {
  await see(url);
} catch (err) {
  if (err instanceof SeeError && err.code === 'rate_limited') {
    await new Promise((r) => setTimeout(r, err.retryAfter * 1000));
  }
}
```

`invalid_url`, `render_failed`, `rate_limited`, `timeout`, `unreachable`. A partially-rendered turntable is **not** an error: you get the frames that worked plus `look.missingViews` naming the angles that did not, because three good views beat a failure.

## Options

| Option | Default | Notes |
| --- | --- | --- |
| `views` | `['three-quarter','front','side','back']` | Up to 6, from the table above |
| `size` | `512` | Square pixels, 128 to 1024 |
| `stats` | `true` | Set false to skip the geometry pass |
| `baseUrl` | `https://three.ws` | Point at another deployment |
| `timeoutMs` | `120000` | Rendering holds a browser for seconds |
| `fetch` | global `fetch` | Inject for tests, proxies, tracing |

## Using MCP instead?

The same capability is an MCP tool, `look_at_model`, on `https://three.ws/api/mcp-studio`. There the frames come back as MCP **image content blocks**, so a multimodal client renders them straight into the conversation and the model sees the model with no glue code at all.

## Related

- [three.ws/docs/3d-vision](https://three.ws/docs/3d-vision): the full surface, REST and MCP
- [`POST /api/3d/look`](https://three.ws/api/3d/look): the endpoint underneath (GET it for a discovery doc)
- [`@three-ws/retarget`](https://www.npmjs.com/package/@three-ws/retarget): retarget animation onto any humanoid GLB

Apache-2.0.

---

Part of [three.ws](https://three.ws), the open-source platform for 3D AI agents. This repository is exported from the [monorepo](https://github.com/nirholas/three.ws/tree/main/packages/see), which is the source of truth: open issues here, and send larger changes there.
