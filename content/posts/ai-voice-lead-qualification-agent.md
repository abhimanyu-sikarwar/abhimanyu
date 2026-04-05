+++
title = "AI Voice Lead Qualification Agent"
date = 2025-06-15
type = "post"
description = "An agentic AI system that autonomously calls sales leads via Sarvam AI text-to-voice, conducts dynamic qualification conversations, and scores leads by interest level"
in_search_index = true
[taxonomies]
tags = ["MCP", "LangGraph", "Python", "AI", "Agents", "Sarvam AI", "WebSocket"]
+++

## Overview

An agentic AI system built with MCP (Model Context Protocol) and LangGraph that autonomously calls sales leads, conducts dynamic qualification conversations via voice, and scores them by interest level — enabling sales teams to focus only on high-intent prospects.

## Tech Stack

- **Orchestration**: MCP, LangGraph
- **Voice**: Sarvam AI text-to-voice
- **Backend**: Python
- **Communication**: WebSocket, REST APIs
- **Intelligence**: LLMs for dynamic conversation flow

## How It Works

The system takes a list of sales leads and autonomously initiates voice calls using Sarvam AI's text-to-voice capabilities. During each call, the LangGraph-powered agent conducts a natural qualification conversation — asking about needs, budget, timeline, and intent — adapting its questions based on the lead's responses.

After each call, the agent scores the lead on interest level and produces a structured qualification summary. Sales teams receive a prioritized list of leads ranked by likelihood to convert, along with conversation transcripts and key takeaways.

## Key Features

- **Autonomous Calling**: Initiates and manages voice calls without human intervention
- **Dynamic Conversations**: LLM-driven dialogue that adapts to each lead's responses in real-time
- **Lead Scoring**: Automated interest-level scoring based on conversation signals
- **MCP Integration**: Uses Model Context Protocol for structured tool-use and agent coordination
- **WebSocket Streaming**: Real-time audio streaming for low-latency voice interactions
