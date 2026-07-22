+++
title = "AI Voice Lead Qualification Agent"
date = 2025-06-15
type = "post"
description = "An agentic AI system that calls sales leads via Sarvam AI text-to-voice, runs qualification conversations, and scores leads by interest level"
in_search_index = true
[taxonomies]
tags = ["MCP", "LangGraph", "Python", "AI", "Agents", "Sarvam AI", "WebSocket"]
+++

## Overview

An agentic AI system built with MCP (Model Context Protocol) and LangGraph that calls sales leads on its own, runs a qualification conversation over voice, and scores each lead by interest level. The sales team only talks to people who already sound like buyers.

## Tech stack

- Orchestration: MCP, LangGraph
- Voice: Sarvam AI text-to-voice
- Backend: Python
- Communication: WebSocket, REST APIs
- Conversation flow: LLMs

## How it works

The system takes a list of sales leads and places voice calls using Sarvam AI text-to-voice. During each call, the LangGraph agent asks about needs, budget, timeline, and intent, and adapts its follow-up questions to what the lead says.

After each call, the agent scores the lead's interest and writes a structured qualification summary. The sales team gets a list ranked by likelihood to convert, with the transcript attached to each entry.

## Features

- Places and manages voice calls with no human on the line
- LLM-driven dialogue that adapts to each lead's responses
- Automated interest scoring from conversation signals
- MCP for structured tool use and coordination between agents
- WebSocket audio streaming to keep voice latency low
