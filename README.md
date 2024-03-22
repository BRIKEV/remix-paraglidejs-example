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


