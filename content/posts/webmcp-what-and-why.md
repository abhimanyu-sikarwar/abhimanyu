+++
title = "What is WebMCP and why does it matter?"
date = 2026-07-06
type = "post"
description = "WebMCP is a proposed web standard that lets a page register structured tools for AI agents, so an agent calls a function instead of guessing."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "JavaScript", "Web Standards"]
+++

An AI agent working through your website today is doing something faintly ridiculous. It reads the DOM, guesses which element is the search box, types into it, guesses which button submits, clicks, waits, re-reads the page, and tries to work out whether anything happened. Every one of those steps can go wrong, and the failures are silent. This is called actuation: the agent pretends to be a human with a mouse.

[WebMCP](https://github.com/webmachinelearning/webmcp) is a proposed web standard that replaces the guessing. Your page registers named tools with a description and a JSON Schema for their inputs, and the agent calls them. Instead of the agent inferring that a button probably starts a checkout, you tell it: here is a tool called `start_checkout`, this is what it does, these are its parameters.

I've been building [MCP servers](@/posts/mcharge-journey.md) for a while, so my first reaction was that this is MCP for the frontend. That's close but not quite right, and the difference matters enough that I wrote [a separate post about it](@/posts/webmcp-vs-mcp.md).

## What the standard actually gives you

Three things, in the order you'll care about them.

Discovery. A page registers tools by name, so an agent can ask what's available and get back a list instead of a screenshot to interpret. A shopping site might expose `search_products`, `add_to_cart`, and `start_checkout`.

Schemas. Each tool declares its inputs as [JSON Schema](https://json-schema.org/understanding-json-schema/reference), so the agent knows a size parameter accepts exactly `Small`, `Medium`, `Large`, or `Extra Large`, and doesn't invent `medium-large`. This is the part that kills a whole category of hallucination, because the constraint is written down rather than implied by a dropdown.

State. Tools can be registered and unregistered as the page changes, so the agent's list of options reflects where the user actually is. A checkout tool that only exists once there's something in the cart can't be called at the wrong moment.

There are two ways to define tools: a [JavaScript API](@/posts/webmcp-imperative-api.md) for anything with logic behind it, and an [HTML annotation approach](@/posts/webmcp-declarative-api.md) that turns an existing form into a tool with two attributes.

## Why this beats actuation

Reliability is the obvious argument. A multi-step actuation flow gives the agent a chance to misread the page at every step, and the steps compound. A tool call either succeeds with the parameters you defined or fails with an error you wrote.

The less obvious argument is that you keep your site. When an agent actuates, it drives your interface blindly and the user watches a cursor move around. When it calls a WebMCP tool, the tool runs on your page, in your UI, with your branding and your design decisions intact. The user can see what happened. That visibility is doing real work for trust: a form that visibly fills in is a very different experience from a black box that reports success.

There's also a maintenance argument that took me a while to appreciate. Actuation couples the agent to your layout, so redesigning a page can silently break every agent that had learned to use it. WebMCP tools couple to your application logic instead. You can move the button.

## What it's good for

The Chrome team's examples are worth reading in full, but the pattern I keep noticing is that WebMCP pays off wherever your interface encodes knowledge the user doesn't have.

Long structured forms are the clearest case. A support request, a warranty claim, an insurance application: the user knows their problem, not your field taxonomy. A `submit_application` tool can map what the user said into the right fields, including the details that trip up autofill, like whether you want one full name or a first and last name separately.

Complex filtering is the other one. Rental listings, hotel search, used cars. The user has criteria in their head ("three bedrooms, under $4500, ten minutes from an A train") and your site has a dozen filters that express some of that. A search tool that takes those criteria as structured parameters gets there in one call.

Then there are the interactions built for human hands that agents handle badly. Date pickers are the classic example. A `date_pick` tool sidesteps the widget entirely.

## The state of it

WebMCP is a proposal, not a shipped standard. It's in [origin trial](https://developer.chrome.com/origintrials/#/register_trial/4163014905550602241) in Chrome from version 149, discussion is happening [in the open on GitHub](https://github.com/webmachinelearning/webmcp), and the API has already changed once (`navigator.modelContext` became `document.modelContext` in Chrome 150). Treat anything you build now as an experiment that you will revise.

That said, it's a progressive enhancement. Tools sit alongside your existing interface rather than replacing it, so a browser without WebMCP support sees the same site it always did. The cost of trying it is low, and the [local setup takes about a minute](@/posts/webmcp-setup.md).

The rest of this series covers the two APIs, the [cases where WebMCP is worth the effort](@/posts/webmcp-use-cases.md), [writing tools agents can actually use](@/posts/webmcp-best-practices.md), [security](@/posts/webmcp-tool-security.md), [testing](@/posts/webmcp-evals.md), and [the limits](@/posts/webmcp-limitations.md).

Source material for this series: Chrome's [WebMCP documentation](https://developer.chrome.com/docs/ai/webmcp) and the [explainer](https://github.com/webmachinelearning/webmcp). Anything I got wrong is mine.
