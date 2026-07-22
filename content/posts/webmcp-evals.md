+++
title = "Testing WebMCP tools with evals"
date = 2026-07-16
type = "post"
description = "How to test tools whose caller is a language model: failure modes, isolated tool tests, expectedCall assertions, end-to-end journeys, and mid-chain failures."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "Testing", "Evals"]
+++

A [WebMCP](@/posts/webmcp-what-and-why.md) tool has two callers. Your own code, which you can test the way you test anything else, and a language model, which you can't. One prompt has thousands of valid-looking responses, so the question stops being "did this return the right value" and becomes "how often does the model do the right thing". Testing that is what evals are for.

The split I'd draw: anything that doesn't touch the model gets a deterministic test, and anything that depends on the model choosing gets an eval. Both are needed, and skipping the second is how you find out in production that your tool was never called.

## Know how it fails

Before writing tests, it helps to enumerate what can actually go wrong. Take a shopping site where the user wants to add a t-shirt to their cart.

The agent picks the wrong tool, skipping `addToCart` and calling `checkout` directly. That's a description problem, an ambiguity problem, or a state problem. Ask whether the description accurately says what the tool does, whether the name is intuitive, whether the tool is exposed in the current state at all, and whether its schema is close enough to another tool's to cause confusion.

The agent calls tools in the wrong order, running `checkout` before `addToCart`. Look at whether descriptions overlap in a way that hides the sequence, whether the earlier tool's output gives enough context for the next call, and whether your state updates exposed the right tools at the right time. It's also worth asking whether the journey genuinely requires that order, because sometimes the constraint is imagined.

The agent calls the right tool with wrong arguments, adding shoes instead of a t-shirt. That's schema work: clear enums, a description on every property, required parameters actually marked required, and descriptions that say how to map what the user said onto your structured values.

The tool returns the wrong thing, like the cart total when the user asked what's in the cart. Check the underlying logic with a deterministic test, confirm the UI state updated, and look at whether the output is formatted for a model to consume rather than for a human to read. Overly verbose output is its own failure, because the fact the model needed is buried.

And tools fail the way all JavaScript fails. Handle runtime errors, report them back to the agent legibly, check that your dependencies are healthy, and make errors distinguishable so a model can tell a temporary problem worth retrying from a permanent one.

## Test tools in isolation first

If an agent can't work out which tool to call for "I'd like a small pizza", it has no chance at a multi-step journey. Isolated tests let you fix schemas and descriptions before you ever run a full simulation.

A single case looks like this:

```json
{
  "messages": [
    { "role": "user", "content": "I'd like a small pizza." }
  ],
  "expectedCall": [
    {
      "functionName": "set_pizza_size",
      "arguments": { "size": "Small" }
    }
  ]
}
```

`expectedCall` makes this a deterministic, rule-based assertion over a probabilistic system: the model can phrase its reasoning however it likes, but either it called `set_pizza_size` with `Small` or it didn't.

One subtlety that's easy to miss. If your tools are tied to component lifecycles, they're only registered in certain states, so an eval has to supply the full tool list for the state you're testing. Evaluating `set_pizza_size` in isolation, with none of the sibling tools present, tests a situation that never occurs. Give the model the same menu it would really see: `add_topping`, `set_pizza_size`, `set_pizza_style`, and the rest. A real agent may have other tools too, but yours are what you can control.

You can also trigger a tool directly with `document.modelContext.executeTool(...)`, which is what makes the deterministic half possible.

## Two kinds of test

Deterministic tests cover everything that doesn't involve the model. Verify the tool's logic, confirm dependencies were called correctly, check that the interface updated and any side effects happened, confirm the returned value matches what you expect, and validate parameters. Mock the dependencies, the same as any integration test. If your tool uses a `SearchComponent`, pass a mock and simulate the environment the tool runs in.

Probabilistic tests cover the model's judgment. Build datasets with both direct queries and ambiguous ones, because they test different things. "Add pepperoni to my pizza" tests baseline execution. "I want all of the meat on my pizza" tests whether the model knows to use `add_topping` and which toppings count as meat. The second kind is where real users live.

Dependencies between tools deserve their own cases. A coffee shop supporting "reorder the same coffee I had last month" needs `get_order_history` to run before `order_product`, because the second one gets its `item_id` from the first. Mock the history service to return a known product ID, then check that the model called the history tool at all. If it skips that step, it's ordering something it guessed.

## End-to-end journeys

Once individual tools behave, test that the sequence holds. A user asks:

> "I am looking to buy a black jacket and a pair of jeans. Could you provide a breakdown of the materials used?"

The journey is: navigate to the category, then for each item find it and get its details. The two items can be handled in either order, but within each item the search has to precede the details lookup. Evals can express exactly that:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "I am looking to buy a black jacket and a pair of jeans. Could you provide a breakdown of the materials used?"
    }
  ],
  "expectedCall": [
    {
      "functionName": "navigate_to_category",
      "arguments": { "category": "clothes" }
    },
    {
      "unordered": [
        {
          "ordered": [
            { "functionName": "search_clothes", "arguments": { "query": "black jacket" } },
            { "functionName": "get_product_details", "arguments": { "productId": "JACKET002" } }
          ]
        },
        {
          "ordered": [
            { "functionName": "search_clothes", "arguments": { "query": "jeans" } },
            { "functionName": "get_product_details", "arguments": { "productId": "JEANS001" } }
          ]
        }
      ]
    }
  ]
}
```

The `ordered` and `unordered` nesting is what makes this usable. Asserting a single rigid sequence would fail every time the model picked jeans first, and you'd spend your time investigating passes.

## Failures in the middle of a chain

The case I'd make sure to cover: a tool fails partway through a sequence and the agent carries on anyway.

A user orders a pizza with a promo code. The chain runs `start_pizza_creator`, `set_pizza_style`, `set_pizza_size`, `start_checkout`, `add_discount_coupon`, `complete_checkout`. If `add_discount_coupon` fails and the agent proceeds, the order completes at full price and everything looks successful. The user finds out when the charge appears.

You can test this without a model in the loop. Execute the sequence manually up to the state where you expect the failure, in this case just after `start_checkout`, then evaluate `add_discount_coupon` on its own. What you're checking is whether the failure is reported in a way that stops the chain rather than getting swallowed. This is also an argument for the [error design](@/posts/webmcp-best-practices.md) I keep coming back to: an error a model can't distinguish from success is worse than a crash.

If evals are new to you, web.dev has a good introduction to [evaluation-driven development](https://web.dev/learn/ai/evaluation-driven-development), and this post follows Chrome's [evals guidance for WebMCP](https://developer.chrome.com/docs/ai/webmcp).
