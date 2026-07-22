+++
title = "Building mCharge: an AI assistant for India's doctors"
date = 2026-07-22
type = "post"
description = "Why I started building an AI assistant for India's doctors, and how it went from HTML mockups to a Play Store app with its own MCP server in six weeks."
in_search_index = true
[taxonomies]
tags = ["MCP", "AI", "Healthcare", "TypeScript", "React Native", "Cloudflare", "Agents"]
+++

For the past six weeks I've been building [mCharge](https://mcharge.in), an AI assistant for India's doctors. It writes clinical notes and discharge summaries, reads photos of lab reports and ECGs, checks drug doses and interactions, and explains a diagnosis in the patient's own language. This post is the story of why I started it and how it took shape.

## Why doctors

I'd spent the previous couple of years on agentic systems: a production reporting assistant at work, MCP side projects at home. Somewhere in there I went down a rabbit hole of "AI in medicine" content — a hospitalist walking through his day with AI, a TV report on AI scribes and their privacy tradeoffs, a GP listing the ways he uses AI to claw back admin time. I ended up merging notes from all three into one backlog, and a pattern stood out: the thing doctors want most is not diagnosis, it's escape from paperwork.

Every tool chasing that demand assumes a Western clinic. They integrate with an EHR, run on a desktop, price in dollars. Most of India's doctors work from a phone. Many clinics have no EHR at all: notes live on paper, lab reports arrive as photos on WhatsApp, and the doctor is the database. An assistant for that reality has to read a photographed lab panel, work on a mid-range Android phone, and answer in the language the patient speaks. Nobody was building that, so I did.

## Week one: mockups to a real app

The first version wasn't a medical app, just plain HTML pages mocking up a chat workspace: projects, skills, settings, history. Within the first day the copy shifted from "AI Workspace" to clinical language, and the mockups became the spec.

The next day I started the real mobile app in Expo and React Native. The backend followed on June 16, and the first backend commit says a lot about the domain: SSRF protection, vault encryption, and MongoDB before any feature work. When the data is health data, security is not a later milestone.

The rest of June was the core loop. PDF rendering and inline markdown landed on June 22, per-patient records and a patient roster on June 23. On June 24 I wrote regulatory groundwork documents, working out where the product sits under India's CDSCO rules and how to stay on the decision-support side of the line. The landing page went up June 25. On July 3 the product got its name, and `com.asikarwar.mcharge` became the package id.

## Teaching it to read

The chat model I use is text-only, which forced an architecture decision I ended up liking. When a doctor attaches a photo of a lab panel or an ECG, the image routes to a separate vision model that transcribes the document: values, units, handwriting, tables, stamps. The chat model then reasons over the transcription. Extractions are cached by content hash, so re-sending the same photo costs nothing.

PDFs take a cheaper path. A born-digital lab report has a text layer, so the backend extracts it locally and no vision model is involved. Scanned PDFs have no text layer; those go through the image pipeline instead.

## The rules that shaped it

One principle from that original backlog survived every rewrite: AI augments, it does not decide. It shows up as hard rules in the product.

The model never does dose or score arithmetic. eGFR, CHA₂DS₂-VASc, Wells, MELD 3.0, and the rest are deterministic calculators with typed inputs; the assistant calls them as tools and reports the result. An LLM that is right about a creatinine clearance 98% of the time is a hazard, and a calculator is right 100% of the time.

Citations only come from documents the doctor provided (project files, attachments, remembered context), never the open web, and the UI treats them as traceability aids, not proof. Memory is capped, user-visible, and sits behind consent controls, alongside an audit trail and configurable retention. These constraints are what make the tool usable in a clinic at all.

## An MCP server any assistant can use

In July the tool layer grew into something standalone. [clinical-mcp](https://clinical-mcp.mcharge.in/mcp) is a TypeScript Model Context Protocol server that gives any LLM clinical data it should never guess: ICD-10 and ICD-11 lookups, LOINC lab codes, HCPCS, the NPI provider registry, RxNorm drug search, drug–drug interaction checks, and 35 of those deterministic calculators, one tool each. The lookups sit on top of the National Library of Medicine's free Clinical Tables and RxNav services, so it needs no API keys.

It runs two ways: over stdio for Claude Desktop and Claude Code, and as a remote server on Cloudflare Workers using stateless Streamable HTTP. mCharge itself consumes it, but so can any MCP client — point Claude at the endpoint and it can check an interaction or score a CHA₂DS₂-VASc mid-conversation. The same tools are browsable on the site at [mcharge.in/tools](https://mcharge.in/tools): 37 calculators, 23 data tools, and 24 datasets.

This was the part of the build where the side projects paid off. The grocery-ordering MCP experiment taught me how assistants behave as orchestrators; clinical-mcp applies that in a domain where a wrong tool result has real consequences.

## Where it is today

Six weeks and a couple hundred commits in, mCharge is live: an Android app on Google Play with in-app updates, a web app, and the public tools directory. The near-term roadmap is ambient scribing (listen to the consult, draft the note for the doctor to edit), evidence-linked answers that flag anything not grounded in a guideline, and handoff digests that summarize what changed in a chart overnight.

If you're a doctor in India, or you know one buried in paperwork, I'd like to hear what would actually help: [mcharge.in](https://mcharge.in), or [reach me here](@/contact.md).
