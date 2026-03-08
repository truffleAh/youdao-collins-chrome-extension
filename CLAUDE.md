# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Chrome extension that provides English word translation using Collins dictionary definitions. The extension supports word selection translation, has a popup search interface, and integrates with Shanbay (扇贝) vocabulary collection. Data is scraped from Youdao Dictionary website.

## Common Commands

### Development
```bash
npm run dev          # Development build with watch mode
npm run dev:static   # Static development server with hot reload
```

### Building
```bash
npm run build        # Production build (cleans build dir, copies assets, runs webpack)
```

### Testing & Code Quality
```bash
npm test             # Run Jest tests
npm run test:watch   # Jest with watch mode
npm run flow         # Flow type checking
```

## Architecture

### Core Components

1. **Content Scripts** (`src/content_script.js`)
   - Runs in web pages to detect word selection
   - Controls when to show translation (ALWAYS, DOUBLE_CLICK, WITH_KEY)
   - Renders translation UI using React components

2. **Event Page** (`src/event_page.js`)
   - Background service worker (Chrome Extension v3)
   - Scrapes Youdao dictionary pages via HTTP requests
   - Handles message passing from content scripts
   - Integrates with Shanbay API for vocabulary collection

3. **Popup UI** (`src/popup.js`)
   - Quick search interface accessible via Ctrl+Q
   - Allows manual word lookup

4. **Options Page** (`src/options.js`)
   - Configuration for translation modes
   - Shanbay OAuth integration

### Data Flow

1. User selects word on webpage
2. Content script sends message to event page
3. Event page fetches Youdao dictionary page
4. Parses HTML using custom Cheerio fork (cheerio-without-node-native)
5. Returns parsed definition to content script
6. UI displays translation using React components

### Key Libraries

- **React 15.4.2** - UI components with inline styles (no external CSS to avoid page conflicts)
- **Cheerio without node-native** - Server-side DOM parsing for cross-domain requests
- **Jest** - Testing framework
- **Flow** - Static type checking

### Build System

- **Webpack 2.3.2** - Bundles multiple entry points:
  - popup
  - content_script
  - event_page
  - options_page
  - words_page
- **Babel** - Transpiles ES6/React to ES5
- Asset copying preserves HTML templates and icons

## Chrome Extension Specifics

### Manifest v3 Configuration
- Service worker background (not persistent)
- Content scripts with `<all_urls>` access
- Permissions for storage, tabs, and specific APIs (Youdao, Shanbay)
- Keyboard shortcut: Ctrl+Q (Mac: MacCtrl+Q)

### Message Passing
Uses Chrome extension messaging API for communication between:
- Content scripts ↔ Event page
- Popup ↔ Event page

## Development Notes

### Inline Styling
All UI uses inline styles to avoid conflicts with web page CSS. React components are self-contained with their styles.

### Cross-Origin Requests
The event page can make cross-domain requests (e.g., to dict.youdao.com) due to Chrome extension permissions, allowing direct page scraping instead of API limits.

### Testing
Tests are located in `/src/__tests__/` and use Jest. Focus is on parsing functionality.

### Configuration
- Shanbay OAuth settings in `src/lib/config.js`
- Extension options stored in Chrome storage