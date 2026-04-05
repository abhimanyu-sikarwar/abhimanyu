+++
title = "AI Grocery Orchestration with MCP"
date = 2025-05-10
type = "post"
description = "A multi-agent grocery ordering system using Model Context Protocol — set dietary preferences in ChatGPT or Claude, get a personalized meal plan, and have ingredients ordered autonomously"
in_search_index = true
[taxonomies]
tags = ["MCP", "LangChain", "LangGraph", "Python", "AI", "Agents", "OpenAI", "Claude"]
+++

## Overview

A multi-agent grocery ordering system powered by Model Context Protocol (MCP). Users set their dietary preferences in ChatGPT or Claude, the AI generates a personalized meal plan, and then autonomously orders the required ingredients from nearby stores based on the user's location.

## Tech Stack

- **Orchestration**: MCP, LangChain, LangGraph
- **LLM Providers**: OpenAI API, Claude API
- **Backend**: Python
- **APIs**: REST APIs for store inventory and ordering

## How It Works

The system is built as a chain of specialized agents coordinated through MCP:

1. **Preference Agent**: Captures the user's dietary preferences, restrictions, and goals through natural conversation in ChatGPT or Claude
2. **Meal Planning Agent**: Generates a personalized weekly meal plan based on preferences, nutritional balance, and variety
3. **Ingredient Agent**: Breaks down the meal plan into a structured grocery list with quantities
4. **Ordering Agent**: Locates nearby stores, checks ingredient availability, and places the order autonomously

Each agent uses tool-calling to interact with external APIs — store inventory lookups, price comparisons, and order placement — while MCP ensures structured communication and state management between agents.

## Key Features

- **Cross-LLM Support**: Works with both ChatGPT and Claude as the user-facing interface
- **Personalized Meal Plans**: Accounts for dietary restrictions, calorie targets, and cuisine preferences
- **Location-Aware Ordering**: Finds and orders from nearby stores based on user location
- **Multi-Agent Coordination**: Specialized agents handle distinct responsibilities via MCP
- **Autonomous End-to-End Flow**: From preference capture to order placement with minimal user intervention
