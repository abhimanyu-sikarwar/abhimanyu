+++
title = "Writing WebMCP tools an agent can actually use"
date = 2026-07-14
type = "post"
description = "Tool strategy, naming, schema design, and reliability for WebMCP: the practices that decide whether an agent picks the right tool and calls it correctly."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "Best Practices", "Web Standards"]
+++

Most [WebMCP](@/posts/webmcp-what-and-why.md) problems aren't JavaScript problems. The code runs fine; the agent just doesn't call it, or calls it with arguments that don't make sense. What decides that is your tool names, descriptions, and schemas, which is an unusual thing for a web developer to have to get right, because these are the first strings you've written whose audience is a language model.

## Plan the tools before writing them

One function per tool. If a tool navigates to a form and fills it in and submits it, an agent has no way to do just the first part, and no way to recover if the third part fails. Split it. The corresponding mistake is splitting too far: if you find yourself asking whether one function could cover several of your tools, it probably should.

Avoid overlap harder than you avoid gaps. Two tools with similar descriptions and similar schemas make the agent guess, and a missing tool produces a clear failure while an ambiguous pair produces a wrong action that looks like it worked.

Register tools when they apply and unregister them when they don't. With the [imperative API](@/posts/webmcp-imperative-api.md) that's `registerTool` plus an `AbortSignal`; with the [declarative API](@/posts/webmcp-declarative-api.md) it's adding or removing `toolname` and `tooldescription`. But keep this simple: for most sites, static registration at load is the right default, and dynamic registration is worth the complexity only when a tool would be actively wrong to call in some states.

There's no hard cap on tool count, but every tool consumes context and slows completion, and the more you register the harder the selection problem gets. This is a case where you should measure rather than guess.

Finally, trust the agent to work out the sequence. Rigid instructions that spell out an exact flow tend to age badly and fight the model's own planning. Describe what each tool does and let it plan.

## Name and describe for a reader who can't see your UI

Distinguish doing from starting. `create-event` should create an event. `start-event-creation-process` should take the user to a form where they can create one. If your name doesn't make that distinction, the agent will pick wrong roughly half the time it matters.

Describe what the tool does and when to use it, in positive terms. Chrome's guidance is explicit about this and it's the opposite of what most of us instinctively write:

Don't write "Don't use this tool for weather." Do write "This tool can create a calendar event, scheduled for a specific date and time."

The negative version invites the model to reason about edge cases you didn't enumerate. The positive version describes a capability, and the limitations follow from it. A well-written description makes the boundaries implicit.

## Don't make the model compute

The rule I'd tattoo on this API: accept raw user input and do the transformation yourself.

If the user says "11:00 to 15:00", take that string. Don't ask the model to work out that it's 240 minutes. Every arithmetic step you push into the model is a step that can be wrong in a way that looks confident.

Declare specific types. String, number, enum, with real constraints. An enum of your four valid sizes is worth more than a paragraph explaining which sizes exist.

Use natural language values rather than internal identifiers. `shipping="Express"` tells the model what it's choosing; `shipping_id=1` tells it nothing, and it will guess. If your backend needs the ID, map it inside your tool. Explaining *why* a choice exists helps too: the model makes better decisions when your parameter descriptions say what the option means, not just that it's allowed.

## Build for the failure cases

Rate limits need graceful failures. Price comparison and repeated searches are legitimate agent behavior, so allow reasonable repetition, and when you do have to refuse, return a meaningful error or tell the user to do it manually. A silent failure is the worst outcome, because the agent will keep going.

Update your interface state when a function completes. Agents read the page to plan their next step, and if your function finishes before your UI reflects it, the agent may act on a stale view. The UI update is part of the tool's contract, not a cosmetic afterthought.

Validate strictly in code and loosely in schema. Schema constraints help the model but aren't guaranteed to hold, so your function still has to check. What matters is what you do when the check fails: return a descriptive error, and the model can correct itself and retry with valid parameters. Return nothing useful, and it can't.

Keep outputs short. The model reads your return value to decide what happens next, so give it the minimum it needs. Chrome suggests a [1.5K character budget per tool output](@/posts/webmcp-tool-security.md), and long outputs both crowd the context window and bury the fact that mattered.

## Test the thing you can't reason about

You can't unit-test whether a model will pick your tool. That needs [evals](@/posts/webmcp-evals.md), and they're less optional here than in most AI work, because tool selection is invisible until it goes wrong in production.

Define the problem like an API contract: input types, output format, constraints. Establish a baseline and an ideal result so you know what you're measuring against. Then decide how you'll judge outputs, whether that's rule-based checks for things like character limits or [an LLM as a judge](https://web.dev/articles/test-llm-capabilities) for the subjective parts.

One anti-pattern to avoid: patching a specific model's mistakes with narrow rules. If your honorific dropdown makes a model pick badly, don't add a rule about honorifics. Make the field optional and let the agent ask the user. Rules written for one model's quirks break when the model updates, and they accumulate into a system nobody can reason about.

Chrome's own [best practices page](https://developer.chrome.com/docs/ai/webmcp/best-practices) is the source for most of this, and they've written [separate guidance on secure tools](https://developer.chrome.com/docs/ai/webmcp/secure-tools) that I'd read before shipping anything that touches user data.
