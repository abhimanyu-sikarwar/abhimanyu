+++
title = "Dart Score — a dart scoring PWA"
date = 2025-08-01
type = "post"
description = "A mobile-first PWA for scoring X01 and Cricket dart games with friends, with an interactive dartboard and automatic win detection"
in_search_index = true
[taxonomies]
tags = ["Next.js", "TypeScript", "PWA", "Radix UI", "Tailwind CSS"]
+++

## Overview

[Dart Score](https://dart.abhimanyusikarwar.com/) is a mobile-first dart scoring app for X01 and Cricket games, built with Next.js and TypeScript. It has two input modes (a quick number grid and an interactive dartboard), bust detection, double-out validation, and full undo.

## Tech stack

- Framework: Next.js 16 with App Router
- Language: TypeScript
- UI: React 19, Tailwind CSS v4, Radix UI primitives
- Theming: next-themes for dark/light mode
- Icons: Lucide React
- PWA: installable, works offline

## Game modes

### X01 (301, 501, 701, custom)

Classic countdown darts: players start from a set score and race to reach exactly zero. Supports double-in and double-out (a double required to start or finish), master-in and master-out (double or triple), and legs and sets in best-of or first-to format.

### Cricket

Standard Cricket scoring with 15 through 20 and Bull. Tracks marks and points across all target numbers.

## Features

- Two input modes: a number grid for speed, or a dartboard for recording exact positions
- Automatic bust detection when a throw would take a player below zero or leave an impossible finish
- Double-out enforcement when the rule is enabled
- Undo for individual darts or entire turns
- Expandable player rows showing all previous throws
- Best-of / first-to matches with configurable legs and sets
- Dark and light mode
- Installable PWA, designed for one-handed use at the dartboard

## Why I built this

Scoring darts on paper is tedious and error-prone, especially with double-out rules and legs/sets in play. Most dart apps are either bloated or get the rules wrong. Dart Score keeps score accurately with as few taps as possible.

Source code on [GitHub](https://github.com/abhimanyu-sikarwar/dart-score). Try it at [dart.abhimanyusikarwar.com](https://dart.abhimanyusikarwar.com/).
