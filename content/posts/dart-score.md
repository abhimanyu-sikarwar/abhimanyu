+++
title = "Dart Score — A Modern Dart Scoring App"
date = 2025-08-01
type = "post"
description = "A mobile-first PWA for scoring X01 and Cricket dart games with friends, featuring an interactive dartboard and smart win detection"
in_search_index = true
[taxonomies]
tags = ["Next.js", "TypeScript", "PWA", "Radix UI", "Tailwind CSS"]
+++

## Overview

[Dart Score](https://dart.abhimanyusikarwar.com/) is a mobile-first dart scoring app for tracking X01 and Cricket games. Built with Next.js and TypeScript, it features dual input modes — a quick number grid and an interactive dartboard — with smart bust detection, double-out validation, and full undo support.

## Tech Stack

- **Framework**: Next.js 16 with App Router
- **Language**: TypeScript
- **UI**: React 19, Tailwind CSS v4, Radix UI primitives
- **Theming**: next-themes for dark/light mode
- **Icons**: Lucide React
- **PWA**: Installable, works offline

## Game Modes

### X01 (301, 501, 701, Custom)
Classic countdown darts. Players start from a set score and race to reach exactly zero. Supports:
- **Double-In / Double-Out**: Require a double to start or finish
- **Master-In / Master-Out**: Require a double or triple
- **Legs and Sets**: Best-of or first-to format for match play

### Cricket
Standard Cricket scoring with 15 through 20 and Bull. Track marks and points across all target numbers.

## Key Features

- **Dual Input Modes**: Quick number grid for speed, or interactive dartboard visualization for precision
- **Smart Win Detection**: Automatic bust detection when a throw would take a player below zero or leave an impossible finish
- **Double-Out Validation**: Enforces the requirement to finish on a double when enabled
- **Full Undo Support**: Remove individual darts or undo entire turns
- **Throw History**: Expandable player rows showing all previous throws
- **Flexible Match Settings**: Best-of / first-to with configurable legs and sets
- **Dark/Light Mode**: Toggle between themes
- **Mobile-First PWA**: Designed for one-handed use at the dartboard, installable on phones

## Why I Built This

Scoring darts on paper is tedious and error-prone, especially with double-out rules and legs/sets. Most dart apps are either bloated or don't handle the rules correctly. Dart Score is focused on doing one thing well — keeping score accurately with minimal taps.

Source code on [GitHub](https://github.com/abhimanyu-sikarwar/dart-score). Try it at [dart.abhimanyusikarwar.com](https://dart.abhimanyusikarwar.com/).
