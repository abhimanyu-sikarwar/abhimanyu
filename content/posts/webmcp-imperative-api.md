+++
title = "Registering WebMCP tools with JavaScript"
date = 2026-07-08
type = "post"
description = "A walkthrough of the WebMCP imperative API: registering and unregistering tools, discovering and executing them, and cross-origin exposure."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "JavaScript", "Web Standards"]
+++

The [imperative API](https://developer.chrome.com/docs/ai/webmcp/imperative-api) is the general-purpose half of [WebMCP](@/posts/webmcp-what-and-why.md). You write plain JavaScript, so a tool can do anything a function can do: change application state, call your API, navigate, or drive a component. If what you want to expose is already an HTML form, [the declarative API](@/posts/webmcp-declarative-api.md) is less work.

Everything hangs off `document.modelContext`. If you're reading older examples that use `navigator.modelContext`, that spelling is deprecated as of Chrome 150.

## Registering a tool

A tool needs a name, a description, an input schema, and a function:

```js
await document.modelContext.registerTool({
  name: 'toggle_layer',
  description: 'Control pizza layers (sauce, cheese). Use "add", "remove", or "toggle".',
  inputSchema: {
    type: 'object',
    properties: {
      layer: { type: 'string', enum: ['sauce-layer', 'cheese-layer'] },
      action: { type: 'string', enum: ['add', 'remove', 'toggle'] },
    },
    required: ['layer'],
  },
  execute: async ({ layer, action }) => {
    await toggleLayer(layer, action);
    return `Performed ${action || 'toggle'} on layer: ${layer}`;
  },
});
```

The schema is [JSON Schema](https://json-schema.org/understanding-json-schema/reference), and the enums are doing more work than they look like they're doing. They're the difference between an agent passing `cheese-layer` and an agent passing `cheese`, `Cheese`, or `the cheese one`.

`execute` returns a string that goes back to the model. Keep it short and factual. The model reads it to decide what to do next, so a confirmation of what changed is useful and a paragraph of prose is not.

For a tool backed by real data, the same shape holds:

```js
await document.modelContext.registerTool({
  name: 'get_order_status',
  description: 'Search orders in a given timeframe. Returns order number, shipping status and location',
  inputSchema: {
    type: 'object',
    properties: {
      timeframe: {
        type: 'string',
        enum: ['today', 'yesterday', 'last_7_days', 'last_30_days', 'last_6_months'],
        description: 'Timeframe for the order lookup.',
      },
    },
    required: ['timeframe'],
  },
  execute: async ({ timeframe }) => {
    // Fetch from your API or database, return the result as a string.
  },
});
```

Note what the enum avoids here. Without it, an agent has to turn "last week" into a date range and hope your backend agrees about what a week is. With it, the agent picks a value you already understand. [Not making the model do arithmetic](@/posts/webmcp-best-practices.md) buys you more accuracy than almost anything else on this page.

## Unregistering

Tools that aren't valid for the current page state should not be visible to the agent. Registration takes an optional `AbortSignal`, so cleanup is the same pattern you already use for event listeners:

```js
const controller = new AbortController();

await document.modelContext.registerTool(addTodoTool, { signal: controller.signal });

// Later, when the tool no longer applies:
controller.abort();
```

In a component-based app this belongs in whatever your framework calls teardown. Angular's [experimental WebMCP support](https://angular.dev/ai/webmcp) wires this to the dependency injection lifecycle so you don't have to think about it.

## Annotations

Tools can carry hints that tell an agent how to treat them:

```js
annotations: {
  readOnlyHint: false,
  untrustedContentHint: true,
}
```

`readOnlyHint` says the tool doesn't change state, which lets an agent skip a confirmation prompt it would otherwise show. `untrustedContentHint` says the tool returns content that came from users or an external source, so the agent should treat the payload as data rather than instructions. Both matter enough that I gave them [their own post on tool security](@/posts/webmcp-tool-security.md).

## Discovering tools

`getTools()` returns the tools the calling document is allowed to see, in alphabetical order:

```js
const [tool] = await document.modelContext.getTools();
console.log(tool);

// {
//   annotations: { readOnlyHint: false, untrustedContentHint: true },
//   description: "Add a new item to the to-do list",
//   inputSchema: '{"type":"object","properties":{"text":{"type":"string"}}}',
//   name: "addTodo",
//   origin: "https://example.com",
//   window: Window {window: Window, self: Window, ...},
// }
```

By default you get same-origin tools only: the ones this document registered, plus any from same-origin documents in the frame tree. To see tools from another origin you have to ask for it explicitly, and the array only accepts secure origins:

```js
// Same-origin tools only
const sameOriginTools = await document.modelContext.getTools();

// Same-origin tools plus tools from a specific partner
const allTools = await document.modelContext.getTools({
  fromOrigins: ['https://partner.org'],
});
```

Two conditions have to hold for a cross-origin tool to show up. You have to list its origin in `fromOrigins`, and that origin has to have exposed the tool to you. Neither side can do it alone.

## Executing a tool

Call a discovered tool with its arguments as a JSON string:

```js
const result = await document.modelContext.executeTool(tool, '{"text": "Buy milk"}');
console.log(result);

// 'Added to-do: Buy milk'
```

The promise resolves with whatever the tool returned, or `null` if the call triggered a navigation. Pending executions can be cancelled the same way registration can:

```js
const controller = new AbortController();

document.modelContext.executeTool(tool, '{"text": "Buy milk"}', {
  signal: controller.signal,
});

controller.abort();
```

`executeTool` is also how you test a tool without a model in the loop, which is the foundation of [running deterministic tests against your tools](@/posts/webmcp-evals.md).

## Reacting to changes

If your code holds a list of tools, it needs to know when that list changes:

```js
document.modelContext.addEventListener('toolchange', (event) => {
  // The available tools have changed.
});
```

This matters most if you're building something agent-side, like the [page agent demo](https://github.com/GoogleChromeLabs/webmcp-tools/tree/main/demos/page-agent), which pulls tools out of an iframe and runs them from a chat interface.

## Cross-origin iframes

Two separate gates control cross-origin access, and both have to open.

The embedding page delegates permission with the `tools` permissions policy:

```html
<iframe src="https://example.com" allow="tools"></iframe>
```

The registering document names who may see the tool:

```js
// https://partner.org
await document.modelContext.registerTool({
  name: 'my_shared_tool',
  description: 'Shared across origins',
  // ...
}, {
  exposedTo: ['https://example.com'],
});
```

Being exposed to an origin doesn't mean that origin automatically sees your tool. It still has to request your origin in `fromOrigins`. The default position is closed on both sides, which is the right default when a tool can take actions on a user's behalf.
