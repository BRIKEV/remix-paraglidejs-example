## Remix Paraglide Examples

This repository contains various examples demonstrating the usage and configuration of remix-paraglide.

Below, we outline the steps to set up a basic environment with Remix and ParaglideJS.

### ParaglideJS

Within your app folder, initialize ParaglideJS using the following command:

```bash
npx @inlang/paraglide-js@latest init
```

You have to answer this when it asks you about the stack.

```
Which tech stack are you using? Vite
```

This sets up ParaglideJS with a folder named `project.inlang` and a file named `project.inlang/settings.json`. In this `settings.json` file, you can specify the languages you need. For instance, let's include Spanish (ES).

```json
{
  "$schema": "https://inlang.com/schema/project-settings",
  "sourceLanguageTag": "en",
  "languageTags": [
    "en",
    "es"
  ],
  "modules": [
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-empty-pattern@latest/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-identical-pattern@latest/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-missing-translation@latest/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-without-source@latest/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/message-lint-rule-valid-js-identifier@latest/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/plugin-message-format@latest/dist/index.js",
    "https://cdn.jsdelivr.net/npm/@inlang/plugin-m-function-matcher@latest/dist/index.js"
  ],
  "plugin.inlang.messageFormat": {
    "pathPattern": "./messages/{languageTag}.json"
  }
}
```

After this, create a new folder named `messages` since it's the name specified in our `settings.json`.

## Remix-paraglidejs

1. Install the required packages:

```bash
npm i --save remix-paraglidejs @inlang/paraglide-js-adapter-vite
```

2. Modify vite.config.ts.

```ts
import { vitePlugin as remix } from "@remix-run/dev";
import { installGlobals } from "@remix-run/node";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
/* --- INCLUDE THIS --- */
import { paraglide } from "@inlang/paraglide-js-adapter-vite";
/* ----------------- */

installGlobals();

export default defineConfig({
  plugins: [
    remix(),
    tsconfigPaths(),
    /* --- INCLUDE THIS --- */
    paraglide({
      project: "./project.inlang", //Path to your inlang project 
      outdir: "./paraglide", //Where you want the generated files to be placed
    }),
    /* ----------------- */
  ],
});
```

3. Include your messages folder with two files like these ones:

`messages/en.js`
```json
{
  "title": "Remix web example",
  "description": "This is a simple example of how to use Remix to create a web app."
}
```

`messages/es.js`
```json
{
  "title": "Remix web de ejemplo",
  "description": "Este es un ejemplo de una web de remix"
}
```

4. Run `npm run dev` to generate the paraglide directory.

5. Modify the Remix entry files [entry.client.tsx](https://remix.run/docs/en/main/file-conventions/entry.client) and [entry.server.tsx](https://remix.run/docs/en/main/file-conventions/entry.server). If you don't see these files, use `npx remix reveal`.

```tsx
// entry.client.tsx
import { RemixBrowser } from "@remix-run/react";
import { startTransition, StrictMode } from "react";
import { hydrateRoot } from "react-dom/client";
/* --- INCLUDE THIS --- */
import { hydrateLang } from 'remix-paraglidejs/client';
import { availableLanguageTags, setLanguageTag } from "<YOUR_PARAGLIDE_DIR>/runtime";
/* ----------------- */

startTransition(() => {
  /* --- INCLUDE THIS --- */
  const lang = hydrateLang('language-tag', availableLanguageTags);
  setLanguageTag(lang);
  /* ----------------- */
  hydrateRoot(
    document,
    <StrictMode>
      <RemixBrowser />
    </StrictMode>
  );
});
```

```tsx
// entry.server.tsx
import { PassThrough } from "node:stream";

import type { AppLoadContext, EntryContext } from "@remix-run/node";
import { RemixServer } from "@remix-run/react";
import { isbot } from "isbot";
import { renderToPipeableStream } from "react-dom/server";
import {
  createReadableStreamFromReadable,
/* --- INCLUDE THIS --- */
  createCookie,
} from "@remix-run/node";
import { setLangServerCookie, getContextLang } from 'remix-paraglidejs/server';
import { setLanguageTag, availableLanguageTags } from "<YOUR_PARAGLIDE_DIR>/runtime";

// language-tag value the same as the one in the entry.client.tsx
export const setLangCookie = createCookie("language-tag", {});
/* ----------------- */

const ABORT_DELAY = 5_000;

export default function handleRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  remixContext: EntryContext,
) {
  return isbot(request.headers.get("user-agent") || "")
    ? handleBotRequest(
        request,
        responseStatusCode,
        responseHeaders,
        remixContext
      )
    : handleBrowserRequest(
        request,
        responseStatusCode,
        responseHeaders,
        remixContext
      );
}

function handleBotRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  remixContext: EntryContext
) {
  return new Promise((resolve, reject) => {
    let shellRendered = false;
    const { pipe, abort } = renderToPipeableStream(
      <RemixServer
        context={remixContext}
        url={request.url}
        abortDelay={ABORT_DELAY}
      />,
      {
        onAllReady() {
          shellRendered = true;
          const body = new PassThrough();
          const stream = createReadableStreamFromReadable(body);

          responseHeaders.set("Content-Type", "text/html");

          resolve(
            new Response(stream, {
              headers: responseHeaders,
              status: responseStatusCode,
            })
          );

          pipe(body);
        },
        onShellError(error: unknown) {
          reject(error);
        },
        onError(error: unknown) {
          responseStatusCode = 500;
          if (shellRendered) {
            console.error(error);
          }
        },
      }
    );

    setTimeout(abort, ABORT_DELAY);
  });
}

function handleBrowserRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  remixContext: EntryContext
) {
  return new Promise((resolve, reject) => {
    let shellRendered = false;
    /* --- INCLUDE THIS --- */
    const lang = getContextLang(remixContext, {
      defaultValue: availableLanguageTags[0],
      availableLanguages: availableLanguageTags,
      // The URL parameter to look for when determining the language
      // for example ($lang)._index.tsx
      urlParam: 'lang',
    });
    setLanguageTag(lang);
    /* ----------------- */
    const { pipe, abort } = renderToPipeableStream(
      <RemixServer
        context={remixContext}
        url={request.url}
        abortDelay={ABORT_DELAY}
      />,
      {
        async onShellReady() {
          shellRendered = true;
          const body = new PassThrough();
          const stream = createReadableStreamFromReadable(body);

          responseHeaders.set("Content-Type", "text/html");
          /* --- INCLUDE THIS --- */
          await setLangServerCookie(lang, responseHeaders, setLangCookie);
          /* ----------------- */

          resolve(
            new Response(stream, {
              headers: responseHeaders,
              status: responseStatusCode,
            })
          );

          pipe(body);
        },
        onShellError(error: unknown) {
          reject(error);
        },
        onError(error: unknown) {
          responseStatusCode = 500;
          if (shellRendered) {
            console.error(error);
          }
        },
      }
    );

    setTimeout(abort, ABORT_DELAY);
  });
}
```

**Remix route usage**

Once you have these files modified you can include your route for example the `_index.tsx` as `($lang)._index.tsx`, and use Paraglide.

```tsx
import { Link } from '@remix-run/react';
import * as m from '<YOUR_PARAGLIDE_DIR>/messages';

export default function Index() {
  return (
    <div style={{ fontFamily: "system-ui, sans-serif", lineHeight: "1.8" }}>
      <h1>{m.title()}</h1>
      <p>{m.description()}</p>
      <ul>
        <li>
          <Link
            to="/en"
            reloadDocument
          >
            EN
          </Link>
        </li>
        <li>
          <Link
            to="/es"
            reloadDocument
          >
            ES
          </Link>
        </li>
      </ul>
    </div>
  );
}
```

Check out more [examples](https://github.com/BRIKEV/remix-paraglidejs/tree/main/examples) to review how to use links without reloadDocument or a page without the `($lang)`.

*Recommendation:* Install [Sherlock extension](https://marketplace.visualstudio.com/items?itemName=inlang.vs-code-extension).

