+++
title = "Export Claude Chat to Markdown"
date = 2025-11-01
type = "post"
description = "A browser extension to export Claude AI conversations in multiple formats: Markdown, Styled HTML, and PDF"
in_search_index = true
[taxonomies]
tags = ["Extension", "Browser", "AI", "Tools"]
+++

## Overview

A browser extension that enables users to export their Claude AI conversations in multiple professional formats. This tool makes it easy to save, share, and archive your AI interactions with beautiful formatting and syntax highlighting.

## Features

- **Multiple Export Formats**: 
  - **Markdown (.md)**: Clean, portable markdown files
  - **Styled HTML (.html)**: Professional HTML with Prism.js syntax highlighting and custom CSS
  - **PDF Export**: Print-ready HTML that opens in a new window for easy PDF generation
- **One-Click Export**: Download any Claude conversation with a single click
- **Bulk Export**: Export multiple conversations at once from chat history
- **Format Preservation**: Maintains conversation structure, code blocks, and formatting
- **Syntax Highlighting**: Automatic code syntax highlighting for multiple languages (Python, JavaScript, SQL, Go, Bash, and more)
- **Customizable Options**: 
  - Choose between all messages or assistant-only responses
  - Include or exclude Claude's thinking blocks
  - Preferences are saved automatically
- **GitHub Gist Integration**: Create and update GitHub Gists directly from conversations
- **Cross-Browser Support**: Works on both Chrome and Firefox

## Installation

### Chrome
1. Download the extension from the [Chrome Web Store](https://chromewebstore.google.com/detail/claude-to-markdown/hpmejedgaglmmbogjgmbphhdopeaglfe?authuser=0&hl=en)
2. Click "Add to Chrome"
3. Grant necessary permissions

### Firefox
1. Download the extension from [Firefox Add-ons](https://addons.mozilla.org/en-GB/firefox/addon/claude-export-to-markdown/)
2. Click "Add to Firefox"
3. Grant necessary permissions

## Usage

### Quick Export from Conversation Page

1. Navigate to any Claude conversation at claude.ai
2. Look for the "Download" button added by the extension (next to the Share button)
3. Click to open the export options menu
4. Choose your preferred format:
   - **Download MD**: Plain markdown file
   - **Download HTML**: Styled HTML with syntax highlighting
   - **Download PDF**: Opens print-ready HTML in new window (use Ctrl+P or Cmd+P to save as PDF)
5. Configure message filtering and thinking block options
6. The file will be downloaded or opened automatically

### Bulk Export from Chat History

1. Navigate to your Claude chat history page
2. Select one or more conversations using the checkboxes
3. Click the export button that appears in the toolbar
4. Monitor progress in the toast notification at bottom-right
5. Files will download automatically with chat titles as filenames
6. Click the X button on the toast to cancel the export at any time

### Extension Popup

1. Click the extension icon in your browser toolbar
2. View the captured conversation in markdown format
3. Use "Refresh Page" to capture the latest data
4. Create or update GitHub Gists for easy sharing

### GitHub Gist Integration

1. Click the settings icon in the extension popup
2. Add your GitHub Personal Access Token
   - Create token at: https://github.com/settings/personal-access-tokens/new
   - Required scope: `gist`
3. Click "Create Gist" or "Update Gist" in the popup
4. Gist will open automatically in a new tab

## Export Format Details

### Markdown Format
- Clean, portable text format
- Preserves conversation structure
- Code blocks with language tags
- Compatible with all markdown editors

### Styled HTML Format
- Professional styling with custom CSS
- Prism.js syntax highlighting for code blocks
- Responsive tables with proper borders and spacing
- Optimized typography for readability
- Can be opened directly in any browser

### PDF Export
- Print-optimized HTML formatting
- Custom CSS for clean, professional output
- Removes browser headers/footers for cleaner PDFs
- Opens in new window for easy browser-based PDF saving
- Maintains all formatting including code highlighting

## Technical Details

The extension uses:
- **Content scripts** to inject download functionality into Claude.ai pages
- **Background scripts** for message handling and storage management
- **Markdown conversion** with proper formatting and code block handling
- **HTML generation** with Prism.js integration for syntax highlighting
- **PDF generation** with custom CSS and print optimization
- **Browser APIs** for file downloads and tab management
- **Chrome Storage API** for data persistence and preference management
- **Fetch API interception** to capture conversation data in real-time

### Key Components

- `markdown-builder.js`: Core markdown generation utilities
- `pdf-generator.js`: PDF export with custom CSS support
- `styled-html-generator.js`: Enhanced HTML generation with Prism.js
- `download-button.js`: In-page download button with format options
- `bulk-export.js`: Bulk export functionality for chat history

## Privacy

- **No data is sent to external servers** (except when creating GitHub Gists, which requires your explicit action)
- **All processing happens locally** in your browser
- **Conversations are only saved** when you explicitly export them
- **GitHub tokens are stored securely** in Chrome local storage
- **HTTPS-only communication** with GitHub API

## Source Code

The project is open source and available on GitHub at [https://github.com/abhimanyu-sikarwar/claude-extension](https://github.com/abhimanyu-sikarwar/claude-extension). Contributions are welcome!

## License

Licensed under Apache License 2.0

## Support

For issues, feature requests, or contributions, please visit the [project repository](https://github.com/abhimanyu-sikarwar/claude-extension).

## Recent Updates

### Latest Features (November 2025)
- ✨ **PDF Export**: Generate print-ready PDFs with custom styling
- ✨ **Styled HTML Export**: Professional HTML with Prism.js syntax highlighting
- 🎨 **Custom CSS Support**: Beautiful formatting for all export types
- 💡 **Syntax Highlighting**: Support for multiple programming languages
- 📋 **Enhanced Tables**: Properly formatted tables in all export formats
- 🔧 **Print Optimization**: Clean PDF output without browser headers/footers
