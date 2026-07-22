+++
title = "Setting up WebMCP for local development"
date = 2026-07-07
type = "post"
description = "How to enable WebMCP in Chrome, register your first tool, and use the inspector extension to see whether an agent can actually call it."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "JavaScript", "Chrome"]
+++

Getting [WebMCP](@/posts/webmcp-what-and-why.md) running locally takes about a minute. Getting a tool that an agent reliably calls takes longer, which is why the second half of this post is about the inspector extension rather than the setup.

## Enable the flag

WebMCP is behind a flag for local development:

1. Open `chrome://flags/#enable-webmcp-testing`
2. Set it to Enabled
3. Relaunch Chrome

That's the whole setup. For a real origin you'd register for the [origin trial](https://developer.chrome.com/origintrials/#/register_trial/4163014905550602241) instead, available from Chrome 149, which gives you a token to serve with your pages so real users' browsers enable the API. Chrome has [a guide to origin trials](https://developer.chrome.com/docs/web-platform/origin-trials) if you haven't used one before.

Two requirements will bite you before your first tool registers.

WebMCP only works in [origin-isolated](https://web.dev/articles/origin-agent-cluster#limitations) documents, which keeps the document's origin stable for the lifetime of a tool. If your site sets `Origin-Agent-Cluster: ?0` to use `document.domain`, the API is disabled outright. That header is a legacy pattern anyway, but plenty of older sites still send it.

The APIs are also gated by a `tools` [permissions policy](https://developer.chrome.com/docs/privacy-security/permissions-policy) that defaults to `self`. Top-level pages and same-origin frames can register tools; cross-origin iframes cannot, unless the embedding page opts them in with `allow="tools"`.

## Register something

The smallest useful tool looks like this:

```js
await document.modelContext.registerTool({
  name: 'set_pizza_size',
  description: 'Set the pizza size directly.',
  inputSchema: {
    type: 'object',
    properties: {
      size: {
        type: 'string',
        enum: ['Small', 'Medium', 'Large', 'Extra Large'],
        description: 'The specific size name.',
      },
    },
    required: ['size'],
  },
  execute: async ({ size }) => {
    applySize(size);
    return `Pizza size set to ${size}`;
  },
});
```

A name, a description, a schema for the inputs, and a function that does the work and returns a string. The [imperative API post](@/posts/webmcp-imperative-api.md) covers the rest of the surface: discovering tools, executing them, unregistering with `AbortSignal`, and cross-origin exposure.

If what you want to expose is already an HTML form, you don't need any of this. Two attributes on the `<form>` element will do it, which is [the declarative API](@/posts/webmcp-declarative-api.md).

One version note that will save you confusion when reading older examples: this used to be `navigator.modelContext`, which is deprecated as of Chrome 150. Use `document.modelContext`.

## Check it with the inspector

Registering a tool is easy. Knowing whether an agent will pick it up, understand its schema, and call it with sensible arguments is the actual work, and you can't tell by reading your own code.

The [Model Context Tool Inspector extension](https://chromewebstore.google.com/detail/model-context-tool-inspec/gbpdfapgefenggkahomfgkhfehlcenpd) gives you an agent to talk to while your page is open. It lists the tools registered on the current page, lets you call them by hand with arguments you choose, shows you the structured output or error your tool returned, and lets you prompt in natural language to see which tool a model actually picks. Prompts go to `gemini-3-flash-preview` by default. This is separate from the Gemini features built into Chrome.

The manual call path is the fastest way to catch schema mistakes: if the browser can't parse your JSON Schema, or your tool returns something unreadable, you find out immediately rather than through a model's confused paraphrase.

The natural language path is where the interesting failures show up. Type what a real user would type, not what your tool is named. If you built `set_pizza_size` and you ask for "a small pizza" and the agent calls `add_topping` instead, your descriptions are the problem, and no amount of correct JavaScript will fix that. [Writing descriptions agents can use](@/posts/webmcp-best-practices.md) is its own skill.

## Read the demos

Google has published working demos for both APIs, and they're small enough to read end to end:

- [Pizza maker](https://github.com/GoogleChromeLabs/webmcp-tools/tree/main/demos/pizza-maker), using the imperative API
- [Flight search in React](https://github.com/GoogleChromeLabs/webmcp-tools/tree/main/demos/react-flightsearch), also imperative
- [Le Petit Bistro](https://github.com/GoogleChromeLabs/webmcp-tools/tree/main/demos/french-bistro), using the declarative API
- [Page agent](https://github.com/GoogleChromeLabs/webmcp-tools/tree/main/demos/page-agent), which retrieves tools from an iframe and runs them in a web chat interface

If you're on Angular, there's [experimental framework support](https://angular.dev/ai/webmcp) that ties tool registration to the dependency injection lifecycle and can turn Signal Forms into tools, which handles the registration and cleanup problem for you.

Once something works locally, the next question is what you should actually expose. I wrote about [where WebMCP earns its keep](@/posts/webmcp-use-cases.md) separately, because the answer isn't "everything".
