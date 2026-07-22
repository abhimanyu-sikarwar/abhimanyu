+++
title = "Export Claude Chat to Markdown"
date = 2025-11-01
type = "post"
description = "A browser extension to export Claude AI conversations as Markdown, styled HTML, or PDF"
in_search_index = true
[taxonomies]
tags = ["Extension", "Browser", "AI", "Tools"]
+++

## Overview

A browser extension that exports Claude AI conversations as Markdown, styled HTML, or PDF. It's for saving, sharing, and archiving your conversations with the formatting and code blocks intact.

## Features

- Three export formats: plain Markdown, styled HTML with Prism.js syntax highlighting, and print-ready HTML for saving as PDF
- One-click export of any conversation
- Bulk export of multiple conversations from chat history
- Keeps conversation structure, code blocks, and formatting
- Syntax highlighting for Python, JavaScript, SQL, Go, Bash, and more
- Export all messages or assistant-only responses, with or without Claude's thinking blocks; preferences are saved
- Create and update GitHub Gists directly from conversations
- Works on Chrome and Firefox

## Installation

### Chrome
1. Download the extension from the [Chrome Web Store](https://chromewebstore.google.com/detail/claude-to-markdown/hpmejedgaglmmbogjgmbphhdopeaglfe?authuser=0&hl=en)
2. Click "Add to Chrome"
3. Grant the requested permissions

### Firefox
1. Download the extension from [Firefox Add-ons](https://addons.mozilla.org/en-GB/firefox/addon/claude-export-to-markdown/)
2. Click "Add to Firefox"
3. Grant the requested permissions

## Usage

### Quick export from a conversation

1. Open any conversation at claude.ai
2. Find the "Download" button the extension adds next to the Share button
3. Click it to open the export menu
4. Pick a format: Markdown file, styled HTML, or print-ready HTML for PDF (use Ctrl+P or Cmd+P to save)
5. Set message filtering and thinking-block options
6. The file downloads or opens automatically

### Bulk export from chat history

1. Open your Claude chat history page
2. Select conversations with the checkboxes
3. Click the export button that appears in the toolbar
4. Watch progress in the toast at bottom-right; click its X to cancel
5. Files download with chat titles as filenames

### Extension popup

1. Click the extension icon in the browser toolbar
2. View the captured conversation as markdown
3. "Refresh Page" re-captures the latest data
4. Create or update a GitHub Gist from here

### GitHub Gist integration

1. Click the settings icon in the popup
2. Add a GitHub personal access token (create one at https://github.com/settings/personal-access-tokens/new with the `gist` scope)
3. Click "Create Gist" or "Update Gist"
4. The Gist opens in a new tab

## Export formats

The Markdown export is a clean portable text file: conversation structure preserved, code blocks tagged with their language, readable in any markdown editor.

The styled HTML export adds CSS and Prism.js syntax highlighting, with proper table borders and spacing. It opens directly in any browser.

The PDF path generates print-optimized HTML in a new window, sized so the browser's own print-to-PDF produces a clean document without browser headers and footers, keeping code highlighting intact.

## Technical details

The extension uses content scripts to inject the download button into claude.ai pages, background scripts for message handling and storage, and fetch API interception to capture conversation data as you browse. Downloads go through the browser's own APIs; preferences persist via the Chrome Storage API.

Main components:

- `markdown-builder.js`: markdown generation
- `pdf-generator.js`: PDF export with custom CSS
- `styled-html-generator.js`: HTML generation with Prism.js
- `download-button.js`: the in-page download button and format menu
- `bulk-export.js`: bulk export from chat history

## Privacy

All processing happens locally in your browser. No data leaves your machine except when you explicitly create a GitHub Gist, and that goes over HTTPS to GitHub only. Conversations are saved only when you export them. GitHub tokens live in Chrome local storage.

## Source code

Open source under Apache License 2.0 at [github.com/abhimanyu-sikarwar/claude-extension](https://github.com/abhimanyu-sikarwar/claude-extension). For issues or feature requests, use the [project repository](https://github.com/abhimanyu-sikarwar/claude-extension).

## Recent updates

November 2025:

- PDF export with print-ready styling, without browser headers and footers
- Styled HTML export with Prism.js syntax highlighting
- Custom CSS support across all export types
- Syntax highlighting for multiple programming languages
- Better table formatting in every export format
