+++
title = "Unusual Flow: an options activity screener"
date = 2025-02-23
type = "post"
description = "A real-time options activity screener for retail traders who want to see where institutional money is flowing"
in_search_index = true
[taxonomies]
tags = ["React", "TypeScript", "WebSocket", "Fintech", "SaaS"]
+++

## Overview

[Unusual Flow](https://unusualflow.com) is a real-time options activity screener for retail traders who want to see where institutional money is flowing. It watches the options market for activity that stands out: large block trades, sweeps, and volume far above the norm for a ticker.

## What it does

The screener scans live options market data and surfaces trades that stand out from the noise:

- Detects large or anomalous trades that may signal institutional positioning
- Tracks aggressive sweep orders that hit multiple exchanges at once
- Flags tickers whose options volume is far above open interest or historical norms
- Aggregates flow data into a read on overall market direction

## Tech stack

- Frontend: React, TypeScript
- Data streaming: WebSocket for real-time market data
- Payments: Stripe for subscription billing
- Analytics: Google Tag Manager

## Features

- Live-streaming options data, updating as trades hit the tape
- Institutional-level trades that most retail platforms don't surface
- A sentiment dashboard aggregating bullish versus bearish flow
- Tiered subscriptions billed through Stripe
- A built-in referral program

## Why I built this

Most retail traders can't see what institutions are doing. The platforms that surface this data either charge a lot or bury the signal in noise. Unusual Flow shows where the big bets are being placed, in an interface fast enough to act on.

Visit [unusualflow.com](https://unusualflow.com) to try it out.
