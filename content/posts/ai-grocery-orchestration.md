+++
title = "AI Grocery Orchestration with MCP"
date = 2025-05-10
type = "post"
description = "A multi-agent grocery ordering system using Model Context Protocol: set dietary preferences in ChatGPT or Claude, get a meal plan, and have the ingredients ordered for you"
in_search_index = true
[taxonomies]
tags = ["MCP", "LangChain", "LangGraph", "Python", "AI", "Agents", "OpenAI", "Claude"]
+++

## Overview

A multi-agent grocery ordering system built on Model Context Protocol (MCP). You set dietary preferences in ChatGPT or Claude, the system generates a meal plan, and it orders the ingredients from a store near you.

## Tech stack

- Orchestration: MCP, LangChain, LangGraph
- LLM providers: OpenAI API, Claude API
- Backend: Python
- APIs: REST APIs for store inventory and ordering

## How it works

The system is a chain of specialized agents coordinated through MCP:

1. A preference agent captures dietary preferences, restrictions, and goals through conversation in ChatGPT or Claude
2. A meal planning agent turns those preferences into a weekly plan
3. An ingredient agent breaks the plan into a grocery list with quantities
4. An ordering agent finds nearby stores, checks availability, and places the order

Each agent calls external APIs for store inventory, prices, and order placement, while MCP carries structured state from one agent to the next.

## Features

- Works with either ChatGPT or Claude as the user-facing interface
- Meal plans account for dietary restrictions, calorie targets, and cuisine preferences
- Orders route to nearby stores based on the user's location
- The flow runs end to end, from preference capture to order placement, with minimal input
