+++
title = "When WebMCP is worth the effort"
date = 2026-07-13
type = "post"
description = "WebMCP pays off where your interface encodes knowledge the user doesn't have: long forms, complex filters, and multi-step flows. Here's how to pick what to expose."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "UX", "Web Standards"]
+++

You can add [WebMCP](@/posts/webmcp-what-and-why.md) tools to any page. That doesn't mean you should. Every tool you register takes up part of the agent's context window and adds one more option it can pick wrongly, so the useful question isn't what you *could* expose but where the gap between what a user wants and what your interface asks for is widest.

Three patterns keep showing up.

## Long structured forms

The user knows their problem. Your form asks for your field taxonomy. Bridging that gap is exactly what a tool call does well.

Chrome's docs walk through a warranty claim, and it's a good example because the user's phrasing contains everything the form needs, just in the wrong shape:

> "Go to the support page and file a warranty claim for my TV. The screen won't turn on. The serial number is XYZ-987. Use my saved details for the rest."

Tools that support this might be `start_claim_process` to get to the right form, `populate_product_details(serial_number, purchase_date)`, `describe_issue(issue_description)`, and `populate_contact_info(name, email, phone)`. The user never had to know your site has a warranty section three levels deep in a support menu.

Timesheets are the same shape with different stakes. A firm where contractors and salaried staff have different billing expectations can annotate one form and let each group's agent fill it correctly:

```html
<form toolname="add-to-timesheet"
      tooldescription="Report billing task and time to add to the timesheet."
      toolautosubmit>
  <label for="date">Date</label>
  <input name="date" type="datetime-local" toolparamdescription="Date of work.">

  <label for="task_category">Task category</label>
  <select id="task_category" name="task_category"
          toolparamdescription="Type of task completed per time block">
    <option value="admin">Admin</option>
    <option value="billing">Billing</option>
    <option value="client">Client meetings or communication</option>
    <option value="development">Development</option>
  </select>

  <label for="minutes_worked">Minutes working on the task</label>
  <input type="number" id="minutes_worked" name="minutes_worked" min="30" max="600"
         toolparamdescription="Minutes worked on this date and task, with a minimum of 30 and maximum of 600.">

  <button type="submit">Update timesheet</button>
</form>
```

Autofill already [cuts form abandonment substantially](https://developer.chrome.com/blog/autofill-insights-2024), and it works on fields a browser recognizes. WebMCP covers the rest: the fields specific to your domain that no autofill heuristic will ever understand.

## Complex filtering

Listing sites are the other clear case. The user has criteria in their head and your site has fifteen filter controls that partially express them.

> "Show me apartments to rent in Brooklyn less than a 10 minute walk from an A train station and under an hour to Tribeca. At least three bedrooms and a dishwasher. Washer and dryer in-unit would be nice. Budget is $4500."

That's a search call plus a filter call:

```js
search({
  location: 'Brooklyn',
  max_price: 4500,
  rooms: 3,
  features: ['dishwasher'],
  optionalFeatures: ['washer-dryer'],
});

apply_filters({
  transit: 'train',
  max_time: '1 hour',
  destination: 'Tribeca',
});
```

Notice the split between required and nice-to-have. A user's real criteria are ranked, and an interface with checkboxes flattens that ranking into a binary. Structured parameters can carry it, which means the agent can return everything matching the hard requirements and mark the ones that also have the optional feature.

Hotel search works the same way, and so does used car search, where the [declarative API](@/posts/webmcp-declarative-api.md) can turn an existing search form into a tool with two attributes and a few field descriptions.

## Multi-step commerce

Shopping is where tool design gets opinionated, because the individual steps are easy and the sequencing is not.

A user planning a kid's birthday party asks their agent to price a shopping list across a few local stores and build a wishlist. For the retailer, useful tools are `search_products` with real parameters (`productType`, `category`, `age`), `add_to_wishlist` so the user reviews before committing, and `refine_search` for price constraints.

The wishlist step is the important one. It gives the user a review point before anything is bought, which is the pattern I'd apply to any tool that spends money: let the agent do the assembly, let the human do the commit. For sensitive actions, the spec also has [`requestUserInteraction()`](https://webmachinelearning.github.io/webmcp/#model-context-client) to ask for confirmation during execution.

Repeat purchases are the easy win in this category. "Reorder the cheese sticks I bought last month" needs `get_order_history(startdate, enddate)` returning products with dates, then `add_to_wishlist(productId, quantity)`. Without the first tool, the second one has no idea what to order, which is a dependency worth [testing explicitly](@/posts/webmcp-evals.md).

## Where I'd skip it

Content pages don't need tools. If the agent's job is to read your article, it can read your article.

Single-action pages that autofill already handles don't need tools either. A newsletter signup with an email field is not a job for a JSON Schema.

Anything that must work without the user present belongs in an [MCP server](@/posts/webmcp-vs-mcp.md), not WebMCP. Tools registered on a page die with the tab.

And if your interface is genuinely simple, adding tools mostly adds ways for an agent to be confused about which one to call. The [best practices post](@/posts/webmcp-best-practices.md) goes into why overlapping tools hurt more than missing ones.

## Working out your own

The framing I've found useful is to write down a critical user journey first: what the user wants, what they'd say to an agent, and every step between that sentence and the outcome. Then look at which of those steps require knowing something about your site rather than something about the user's problem. Those steps are your tools.

If a step only requires knowledge the user already has, the agent probably doesn't need help with it.
