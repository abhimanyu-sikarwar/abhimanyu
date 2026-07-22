+++
title = "What WebMCP can't do yet"
date = 2026-07-17
type = "post"
description = "The constraints worth knowing before you build on WebMCP: no headless calls, no cross-site discovery, refactoring costs, and a spec that's still moving."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "Web Standards"]
+++

I've spent [nine posts](@/posts/webmcp-what-and-why.md) on what [WebMCP](https://github.com/webmachinelearning/webmcp) does. This one is the other side, because the limits shape what you should build more than the features do.

## Tools need an open tab

Tool calls run in JavaScript, so there has to be a browser tab or webview open with your page in it. There's no headless mode, no way for an agent to call your tools while the user is asleep, and no background execution.

This kills a set of use cases people reach for immediately. Scheduled actions, agents working through a task list overnight, anything triggered by a server-side event. All of that belongs in an [MCP server](@/posts/webmcp-vs-mcp.md), and the division is clean enough that I'd treat "does this need to work with nobody present" as the first question when deciding where a capability lives.

It's worth saying that the constraint is also a feature. A tool that only exists while your user has your site open has a naturally small blast radius, which matters given [what prompt injection can do](@/posts/webmcp-tool-security.md).

## Nobody can find your tools

Clients and browsers only learn that your site has tools by visiting it. There's no registry, no discovery protocol, nothing an agent can query to find sites that support the task at hand.

For a user who's already on your site with an agent, that's fine. For the scenario people imagine, where an agent goes off and finds the best price across five retailers, it means the agent has to already know to visit those five. Whether that gets solved by the standard, by convention, or by agents maintaining their own lists is an open question, and it's worth watching if your business depends on being found rather than being used.

## Complex interfaces cost real work

The [declarative API](@/posts/webmcp-declarative-api.md) is nearly free if you have well-built HTML forms with labels and names and required attributes. If your interface is a pile of divs with click handlers, there's nothing for the browser to derive a schema from, and you're either refactoring toward semantic HTML or writing [imperative tools](@/posts/webmcp-imperative-api.md) by hand.

Even with the imperative API, exposing tools forces you to have an answer for application state that many single-page apps have never needed to articulate. Which tools are valid right now? What updates when this one completes? How does a component tear down its registrations? These are answerable, and Angular's [experimental support](https://angular.dev/ai/webmcp) answers some of them for you, but "add WebMCP" is not a one-afternoon task on a large app.

## It's an origin trial, and the API has already moved

WebMCP is a proposal under [active discussion](https://github.com/webmachinelearning/webmcp), available in Chrome through an [origin trial](https://developer.chrome.com/origintrials/#/register_trial/4163014905550602241) from version 149. It is not a ratified standard and no other browser ships it.

The API has already changed once in public: `navigator.modelContext` became `document.modelContext` in Chrome 150. The [character budgets](@/posts/webmcp-tool-security.md) are described as recommendations that may become spec limits. Consent management is [still being designed](https://github.com/webmachinelearning/webmcp/issues/176).

None of that is a reason to stay away, but it is a reason to keep your tool layer thin. If your tools are wrappers over functions you'd have written anyway, a spec change costs you a refactor of the wrapper. If your business logic lives inside `execute` callbacks, it costs you more.

## The model is still the weak link

The thing WebMCP fixes is ambiguity about what your site can do. What it doesn't fix is the model deciding correctly which tool to use, when, and with what arguments. A perfectly specified tool can still be called at the wrong moment by a model that misread the user, which is why [evals](@/posts/webmcp-evals.md) aren't optional and why anything irreversible should still require a human.

Treat WebMCP as removing one large source of error rather than as making agent behavior deterministic.

## Where I've landed

I'd build on it now, in a limited way, for the cases where the payoff is clear: long forms, complex filters, [flows where users routinely get lost](@/posts/webmcp-use-cases.md). I'd keep the tool layer thin, keep anything that must work headlessly in an MCP server, and I wouldn't rewrite an interface just to make it agent-friendly until the standard settles.

It's a progressive enhancement. That's the honest reason to try it early: the downside is bounded, since browsers without support see exactly the site they see today.

If you're starting, the [local setup](@/posts/webmcp-setup.md) takes a minute, and Chrome's [WebMCP docs](https://developer.chrome.com/docs/ai/webmcp) are the authoritative source for everything in this series. You can also [join the origin trial](https://developer.chrome.com/origintrials/#/register_trial/4163014905550602241) or file feedback on the [explainer](https://github.com/webmachinelearning/webmcp), which is where the API is being decided while it's still cheap to change.
