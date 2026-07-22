+++
title = "WebMCP and MCP are not competitors"
date = 2026-07-10
type = "post"
description = "WebMCP does not replace MCP. One exposes your backend to any agent anywhere, the other makes your live web UI callable. Most systems want both."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "Architecture", "Web Standards"]
+++

The first question I had about [WebMCP](@/posts/webmcp-what-and-why.md), and the one Chrome's team says they hear most, is whether it replaces [Model Context Protocol](https://modelcontextprotocol.io/). It doesn't. They solve different problems, and once you see where the line falls, the question stops making sense.

I've built on both sides of it. For [mCharge](https://mcharge.in) I wrote a clinical MCP server that exposes drug interaction checks and 35 deterministic medical calculators, running as a remote server on Cloudflare Workers. That server has nothing to do with any web page. It answers whenever an agent asks, whether the doctor has our app open or not. That's MCP's job, and WebMCP couldn't do it.

## Where the functionality lives

MCP connects an agent to systems: data sources, APIs, workflows. It's transport-agnostic, usually [JSON-RPC](https://www.jsonrpc.org/), and you implement it with an SDK in whatever language your backend speaks. The agent doesn't need a browser, or a user, or a tab. It needs a network connection.

WebMCP connects an agent to a page that's open right now. You implement it in JavaScript or HTML attributes, and the browser mediates between your document and its built-in agent. It's deliberately not a JavaScript port of MCP: it drops server-side concepts like [resources](https://modelcontextprotocol.io/specification/2025-06-18/server/resources#resources) and keeps only what makes sense inside a document.

The analogy Chrome's docs use is a call center versus an in-store expert, and it holds up. The call center answers at any hour from anywhere and can look things up in the system. The in-store expert only exists while you're in the store, but they can see what you're holding.

## What they share

Both replace inference with declaration, which is the actual point of either protocol.

Both give an agent a machine-readable answer to "what can you do", instead of making it work that out from a UI or an undocumented API. Both replace simulated interaction with explicit function calls, so the outcome is predictable rather than dependent on a layout that might change. And both let you state a capability's purpose directly, rather than hoping an agent infers it correctly from a button label.

If you've written an MCP server, none of WebMCP's concepts will surprise you. Names, descriptions, JSON Schema inputs, structured results. The mental model transfers.

## Where they differ

|  | MCP | WebMCP |
|---|---|---|
| Purpose | Data and actions available to agents anywhere, anytime | A live page ready for agent interaction while the user is on it |
| Lifecycle | Persistent (server or daemon) | Ephemeral (tab-bound) |
| Connectivity | Global: desktop, mobile, cloud, web | Browser agents only |
| UI interaction | Headless and external | Browser-integrated, DOM-aware |
| Discovery | Agent-specific registration flows | Registered on the page during the visit |
| Typical use | Background API actions | Navigating and acting on a live UI |

The lifecycle row is the one that decides most architecture questions. MCP tools are there whether or not anyone is looking. WebMCP tools exist only while the tab is open, and the moment the user navigates away, the agent loses access. That's a constraint, and it's also a security property: nothing you expose through WebMCP can be called by an agent that isn't currently on your site with your user.

## Who owns the interface

There's a difference here that took me longer to notice than it should have.

When an agent uses your MCP server, your application is a guest inside the agent. Any UI you render appears within the agent's surface, on its terms, and building that UI means building a second application.

When an agent uses your WebMCP tools, the agent is a guest on your site. Tools run against live session data, cookies, and DOM state that only exist in an open tab. The user sees your interface doing the work. You don't build a second application, and you don't hand your brand over to someone else's chat window.

The practical consequences are worth listing. Calls are fast, because there's no round trip to a remote server. Tools bind to application logic rather than layout, so a redesign doesn't break agent behavior the way actuation does. And you decide what an agent is allowed to do, instead of hoping it finds the right button.

## Using both

For most products the split falls out naturally.

Your MCP server holds the core logic: data retrieval, business rules, background actions. It's platform-agnostic and always available, so an agent can act for your user at three in the morning with no browser involved. That's where I'd put anything that has to work without a session, which for mCharge meant every clinical lookup and calculator.

Your WebMCP tools handle what only makes sense in a live tab: multi-step flows tied to page state, forms that need to be filled in front of the user, filtering against results the user is looking at. They're the contextual layer for when the person is right there.

The failure mode I'd watch for is building the same capability twice with two sources of truth. If a WebMCP tool and an MCP tool both place an order, they should both call the same service, and the WebMCP one should be a thin wrapper that updates your UI afterward. The browser layer is a better interface to your logic, not a second copy of it.
