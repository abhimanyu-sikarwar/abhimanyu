+++
title = "Tool security for WebMCP"
date = 2026-07-15
type = "post"
description = "Prompt injection, annotation hints, cross-origin exposure, and character budgets: what to get right before exposing WebMCP tools that touch user data."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "Security", "Prompt Injection"]
+++

Here's the uncomfortable premise underneath every agentic system: a language model reads instructions and data as the same stream of tokens. It has no structural way to tell your system prompt from a product review someone wrote. That's what makes indirect prompt injection possible, where an attacker plants instructions in content the model will read, and it's why [WebMCP](@/posts/webmcp-what-and-why.md) tool security is a design problem rather than a validation problem.

Some models have layers that resist this. None of them guarantee anything, because models are probabilistic. There are [documented, repeatable injection attacks](https://bughunters.google.com/blog/task-injection-exploiting-agency-of-autonomous-ai-agents) against agentic systems built on current models, and Google reports that [attacks on the web are becoming more common](https://blog.google/security/prompt-injections-web/). Design as if the model can be talked into things, because it can.

## Label untrusted content

Two annotations go on your tool definitions, and they're the cheapest security work available:

```js
annotations: {
  readOnlyHint: true,
  untrustedContentHint: true,
}
```

`untrustedContentHint` marks a tool whose output contains user-generated or externally sourced content. Comments, reviews, uploaded documents, third-party API responses. It tells the agent that this payload is data to be examined rather than instructions to be followed. Any tool that returns something a stranger could have written should carry it.

`readOnlyHint` marks a tool that doesn't change state. That lets an agent skip a confirmation prompt it would otherwise raise, which improves the experience for safe operations and, more importantly, preserves the user's attention for the operations that do change something. If everything prompts, nobody reads the prompts.

Neither hint enforces anything. They're signals that let a well-behaved agent make better decisions, which is most of what a page-side API can do.

## Expose tools deliberately

By default, `document.modelContext.registerTool` exposes your tool to agents only. Other websites and cross-origin iframes can't see it or call it. That default is correct and you should think hard before widening it.

Widening it looks like this:

```js
// https://partner.org
await document.modelContext.registerTool({
  name: 'my_shared_tool',
  description: 'Shared across origins',
  // ...
}, {
  exposedTo: ['https://trusted.com', 'https://example.com'],
});
```

The array accepts secure origins only, and exposure runs both directions: those origins can reach the tool when embedded on your site, and when your site is embedded on theirs. There's a second gate too, since the consuming origin still has to ask for your origin explicitly in `getTools({ fromOrigins })`. Both sides have to agree, as I covered in the [imperative API post](@/posts/webmcp-imperative-api.md).

The question to ask about each origin is not "do I trust this company" but "would I hand this data or this action to this site directly".

Read-only tools leak. A `getFavoriteProducts` tool tells the caller what a user likes, which is information you'd think twice about sharing in any other context. Expose it only to origins you'd share that data with deliberately.

Write tools act. A `postComment` tool posts as your user. You might expose that to a trusted partner; you obviously wouldn't expose it to a site you've never heard of. And the boundary is only as good as your list, so keep it short and review it.

One thing worth knowing about the threat model: Chrome extensions with `host_permission` can query and execute WebMCP tools through content scripts. Those extensions can already run arbitrary JavaScript on your page, so this isn't a new hole, but it does mean the tool boundary isn't the only boundary that matters.

## Keep descriptions and outputs small

This reads like a performance tip and is partly a security one. Agents apply guardrails, and verbose tools run into them. Chrome's current recommendations:

- 500 characters per tool description
- 150 characters per parameter description
- 30 characters per tool name and per parameter name
- 1.5K characters per tool output

The output limit is the one I'd watch. A tool that returns a large blob of user-generated content is both crowding the model's context and maximizing the surface area for anything malicious hidden in that content. Return the minimum the agent needs for its next step.

These numbers vary by agent and are subject to change, so treat them as a starting budget rather than a specification.

## What isn't solved yet

Consent management across parties is [an open discussion](https://github.com/webmachinelearning/webmcp/issues/176), and the spec draft includes [`requestUserInteraction()`](https://webmachinelearning.github.io/webmcp/#model-context-client) so a tool can ask for user input during execution. That's the mechanism I'd use for anything irreversible, though it's early and the shape may change.

For now, the practical position is: assume the model can be manipulated, keep the blast radius of any single tool small, make anything irreversible require a human, and label the data you can't vouch for. If you're building an agent rather than a site, Chrome has [separate security guidance](https://developer.chrome.com/docs/agents/security) for that side.

Sources for this post are Chrome's [tool security guidance](https://developer.chrome.com/docs/ai/webmcp/secure-tools) and [best practices](@/posts/webmcp-best-practices.md).
