+++
title = "Turning HTML forms into WebMCP tools"
date = 2026-07-09
type = "post"
description = "The WebMCP declarative API turns an HTML form into an agent-callable tool with two attributes, plus events and CSS hooks for what happens next."
in_search_index = true
[taxonomies]
tags = ["WebMCP", "MCP", "AI", "Agents", "HTML", "Forms", "Web Standards"]
+++

The [declarative API](https://developer.chrome.com/docs/ai/webmcp/declarative-api) is the part of [WebMCP](@/posts/webmcp-what-and-why.md) I'd reach for first, because most of the interactions worth exposing to an agent are already forms. You add two attributes and the browser derives a tool from markup you've already written, including the JSON Schema, which it builds from your fields.

## Two attributes

```html
<form toolname="createSupportRequest"
      tooldescription="Submits a request for customer support.">
</form>
```

`toolname` names the tool and `tooldescription` says what it does. That's registration. Remove either attribute and the tool is unregistered, which means toggling availability as page state changes is a matter of setting or clearing an attribute.

When an agent calls the tool, the browser focuses the form and fills its fields. The form stays visible the whole time, so the user watches their information land in the right places. This is the property I like most about the declarative API: there's no separate agent surface to trust, just your form filling itself in.

## Describing fields

The browser infers each parameter's description from the field's `<label>`, falling back to `aria-description` if there isn't one. When the label isn't enough, `toolparamdescription` overrides it:

```html
<form toolname="supportRequestTool"
      tooldescription="Submit a request for support."
      action="/submit">

  <label for="firstName">First Name</label>
  <input type=text name=firstName>

  <label for="lastName">Last Name</label>
  <input type=text name=lastName>

  <select name="select" required
          toolparamdescription="Determines what team this request is routed to.">
    <option value="Customer happiness team">Return my purchase.</option>
    <option value="Distribution team">Check where my package is.</option>
    <option value="Website support team">Get help on the website.</option>
  </select>

  <button type=submit>Submit</button>
</form>
```

The select is the interesting one. Its label says nothing useful to an agent about routing, so the attribute supplies the missing intent.

The browser turns that markup into this:

```json
[
  {
    "name": "supportRequestTool",
    "description": "Submit a request for support.",
    "inputSchema": {
      "type": "object",
      "properties": {
        "firstName": { "type": "string" },
        "lastName": { "type": "string" },
        "select": {
          "type": "string",
          "anyOf": [
            { "type": "string", "const": "Customer happiness team", "title": "Return my purchase." },
            { "type": "string", "const": "Distribution team", "title": "Check where my package is." },
            { "type": "string", "const": "Website support team", "title": "Get help on the website." }
          ],
          "enum": [
            "Customer happiness team",
            "Distribution team",
            "Website support team"
          ],
          "description": "Determines what team this request is routed to."
        }
      },
      "required": ["select"]
    }
  }
]
```

Your options became an enum and your `required` attribute became a required property. Writing that schema by hand in the [imperative API](@/posts/webmcp-imperative-api.md) would take a while, and you'd have to keep it in sync with the markup by hand.

This is also an argument for semantic HTML that has nothing to do with agents. A form built with real labels, real `name` attributes, and real `required` flags produces a good schema for free. A form built out of divs and click handlers produces nothing.

## Submitting

By default the user clicks submit themselves. The agent fills the form, the human commits it. For a warranty claim or a purchase, that's the behavior you want.

Add `toolautosubmit` and invoking the tool submits the form and triggers whatever navigation follows:

```html
<form toolautosubmit toolname="search_tool"
      tooldescription="Search the web" action="/search">
  <input type=text name=query>
</form>
```

I'd reserve this for searches and filters, anything the user can undo by trying again. A tool that spends money or files a legal document should make a human press the button.

## Knowing an agent did it

`SubmitEvent` gains an `agentInvoked` boolean, true when the submission came from an agent rather than a person. It also gains `respondWith(Promise)`, which lets you hand the browser a promise whose resolved value is serialized and returned to the model as the tool's output. You have to call `preventDefault()` first to stop the normal submission:

```js
document.querySelector('form').addEventListener('submit', (e) => {
  e.preventDefault();

  if (!myFormIsValid()) {
    if (e.agentInvoked) {
      e.respondWith(myFormValidationErrorPromise);
    }
    return;
  }

  if (e.agentInvoked) {
    e.respondWith(Promise.resolve('Search is done!'));
  }
});
```

The validation branch is the part worth copying. Without `respondWith`, a failed validation looks to the agent like nothing happened, and it has no way to know that the phone number was in the wrong format. With it, the agent gets your error message and can retry with a corrected value. Descriptive errors are how a model self-corrects, and they cost you one string.

## Events and focus styling

The window gets two events. `toolactivated` fires once fields are pre-filled, and `toolcancel` fires if the user cancels the operation or the form is reset. Neither is cancelable, and both carry a `toolName`:

```js
window.addEventListener('toolactivated', ({ toolName }) => {
  console.log(`the tool "${toolName}" execution was activated.`);
});

window.addEventListener('toolcancel', ({ toolName }) => {
  console.log(`the tool "${toolName}" execution was cancelled.`);
});
```

Chrome also applies two pseudo-classes during agent invocation, so users can see which form an agent is working on: `:tool-form-active` on the form, and `:tool-submit-active` on its submit button. They clear when the form submits, the agent cancels, or the user resets it. The browser defaults look like this:

```css
form:tool-form-active {
  outline: light-dark(blue, cyan) dashed 1px;
  outline-offset: -1px;
}

input:tool-submit-active {
  outline: light-dark(red, pink) dashed 1px;
  outline-offset: -1px;
}
```

You can restyle these, and you probably should so they match your design, but don't remove the visual signal. A user needs to see where on the page something is happening on their behalf. Web.dev's [focus guidance](https://web.dev/learn/accessibility/focus) applies here as much as it does to keyboard navigation.

## What this doesn't cover

The declarative API handles form submission well and nothing else. Navigation, state changes, and anything that isn't shaped like a form needs [the imperative API](@/posts/webmcp-imperative-api.md). Plenty of real sites will want both: a JavaScript tool to get the user to the right page, and an annotated form to fill in once they're there.
