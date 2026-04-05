+++
title = "BhashaSethu — A Hindi-Kannada Voice Translator for Travellers"
date = 2025-08-01
type = "post"
description = "A mobile-first PWA that provides live Hindi-to-Kannada voice translation using Sarvam AI, built for travellers navigating South India"
in_search_index = true
[taxonomies]
tags = ["Next.js", "TypeScript", "Sarvam AI", "PWA", "AI", "Translation"]
+++

## Overview

[BhashaSethu](https://local-guide.abhimanyusikarwar.com/) (meaning "Language Bridge") is a mobile-first progressive web app that provides live Hindi-to-Kannada voice translation. Built for travellers who speak Hindi and need to communicate in Karnataka, it uses Sarvam AI's Indic language models for speech-to-text, translation, and text-to-speech — all in a single press-and-hold interaction.

## Tech Stack

- **Framework**: Next.js 16 with App Router
- **Language**: TypeScript
- **UI**: React 19, Tailwind CSS v4, shadcn/ui
- **State Management**: Zustand
- **AI/ML**: Sarvam AI SDK — speech-to-text (Saaras v3), translation (Mayura v1), text-to-speech
- **PWA**: Installable, offline-ready

## How It Works

1. **Press and hold** the mic button to record speech in Hindi or Kannada
2. Sarvam AI transcribes the audio to text using the Saaras speech-to-text model
3. The Mayura translation model converts the text to the target language
4. Text-to-speech plays the translation aloud so the listener can understand

The app caches TTS audio to avoid repeated API calls for frequently used phrases.

## Key Features

- **Voice-First UX**: Press-and-hold recording with automatic transcription, translation, and playback
- **Bidirectional Translation**: Hindi to Kannada and Kannada to Hindi
- **Phrasebook**: Pre-built common phrases with tap-to-speak for quick communication
- **Quick Cards**: One-tap common phrases for everyday situations
- **Translation History**: Recent translations saved for easy access
- **Audio Caching**: Cached TTS responses to reduce API calls and improve speed
- **Dark/Light Mode**: Automatic theme switching
- **Mobile-First PWA**: Installable on phones, designed for one-handed thumb use
- **11 Indian Languages**: Sarvam AI supports Bengali, English, Gujarati, Hindi, Kannada, Malayalam, Marathi, Odia, Punjabi, Tamil, and Telugu

## Why I Built This

Moving to Bangalore as a Hindi speaker, everyday interactions — with auto drivers, shopkeepers, neighbours — often hit a language wall. Existing translation apps feel clunky for quick spoken exchanges. BhashaSethu is designed for exactly this: hold the button, speak, and let the other person hear the translation instantly.

Source code on [GitHub](https://github.com/abhimanyu-sikarwar/local-guide). Try it at [local-guide.abhimanyusikarwar.com](https://local-guide.abhimanyusikarwar.com/).
