+++
title = "BhashaSethu: Hindi-Kannada voice translator"
date = 2025-08-01
type = "post"
description = "A mobile-first PWA for live Hindi-to-Kannada voice translation using Sarvam AI, built for travellers navigating South India"
in_search_index = true
[taxonomies]
tags = ["Next.js", "TypeScript", "Sarvam AI", "PWA", "AI", "Translation"]
+++

## Overview

[BhashaSethu](https://local-guide.abhimanyusikarwar.com/) ("language bridge") is a mobile-first progressive web app for live Hindi-to-Kannada voice translation. It's built for travellers who speak Hindi and need to communicate in Karnataka, using Sarvam AI's Indic language models for speech-to-text, translation, and text-to-speech in a single press-and-hold interaction.

## Tech stack

- Framework: Next.js 16 with App Router
- Language: TypeScript
- UI: React 19, Tailwind CSS v4, shadcn/ui
- State management: Zustand
- AI: Sarvam AI SDK — speech-to-text (Saaras v3), translation (Mayura v1), text-to-speech
- PWA: installable, offline-ready

## How it works

1. Press and hold the mic button to record speech in Hindi or Kannada
2. Sarvam AI transcribes the audio with the Saaras speech-to-text model
3. The Mayura translation model converts the text to the target language
4. Text-to-speech plays the translation aloud for the listener

The app caches TTS audio, so common phrases don't cost repeated API calls.

## Features

- Press-and-hold recording with automatic transcription, translation, and playback
- Works both directions: Hindi to Kannada and Kannada to Hindi
- Phrasebook of common phrases with tap-to-speak
- One-tap quick cards for everyday situations
- History of recent translations
- Cached TTS audio for speed and fewer API calls
- Automatic dark and light mode
- Installable PWA designed for one-handed thumb use
- Sarvam AI supports 11 Indian languages: Bengali, English, Gujarati, Hindi, Kannada, Malayalam, Marathi, Odia, Punjabi, Tamil, and Telugu

## Why I built this

Moving to Bangalore as a Hindi speaker, everyday interactions with auto drivers, shopkeepers, and neighbours often hit a language wall. Existing translation apps feel clunky for quick spoken exchanges. BhashaSethu is built for exactly this: hold the button, speak, and let the other person hear the translation.

Source code on [GitHub](https://github.com/abhimanyu-sikarwar/local-guide). Try it at [local-guide.abhimanyusikarwar.com](https://local-guide.abhimanyusikarwar.com/).
