# WXT Chrome Extension Development Skill

## Skill Identity
**Name:** WXT Chrome Extension Development  
**Version:** 1.0.0  
**Purpose:** Build production-ready Chrome Extensions (Manifest V3) using WXT framework - the Next.js for Web Extensions

## When to Use This Skill
Trigger this skill when the user requests:
- Creating a new Chrome extension with WXT
- Building browser extensions for multiple browsers
- Developing extensions with Vue, React, Svelte, or SolidJS
- Fast development workflow with HMR
- Cross-browser extension development
- Auto-import and TypeScript setup
- Extension with Vite build system
- Keywords: "WXT extension", "web extension framework", "wxt.dev", "browser extension", "manifest v3", "vite extension", "cross-browser extension"

## Core Framework Philosophy
WXT is the **"Nuxt.js for Web Extensions"** - built on Vite, providing:
- **File-based entry points**: Directory structure automatically generates manifest
- **Framework agnostic**: Works with React, Vue, Svelte, SolidJS, and more
- **Lightning-fast HMR**: Instant updates for UI and fast reloads for background/content scripts
- **Auto-imports**: Like Nuxt.js - automatically import browser APIs and utilities
- **Cross-browser**: Single codebase for Chrome, Firefox, Edge, Safari
- **TypeScript-first**: Built for type safety and large projects

## Key Differences from Plasmo

| Feature | WXT | Plasmo |
|---------|-----|--------|
| Build Tool | Vite (faster) | Parcel |
| Framework Support | Any (Vue, React, Svelte, etc.) | React-focused |
| Auto-imports | ✅ Nuxt-style | ❌ |
| HMR Speed | ⚡ Lightning fast | Good |
| Philosophy | Nuxt-like, minimal config | Next.js-like, opinionated |
| Manifest Control | More explicit | More automatic |
| Learning Curve | Slightly steeper | Easier |

**Use WXT when:**
- You want maximum flexibility with frameworks
- Build speed is critical
- You prefer Vite ecosystem
- You need cross-browser support from day 1
- You like Nuxt.js-style development

---

## Project Structure & Setup

### Initialize New Project
```bash
# Using pnpm (recommended)
pnpm create wxt

# Using npm
npm create wxt

# With specific template
pnpm create wxt my-extension --template react
pnpm create wxt my-extension --template vue
pnpm create wxt my-extension --template svelte
pnpm create wxt my-extension --template solid

# Development
pnpm dev

# Build for production
pnpm build

# Build for specific browser
pnpm build:firefox
pnpm build --browser firefox

# Zip for submission
pnpm zip
```

### Standard WXT Project Structure
```
my-extension/
├── public/                  # Static assets (icons, images)
│   └── icon/
│       ├── 16.png
│       ├── 32.png
│       ├── 48.png
│       └── 128.png
├── entrypoints/             # Extension entry points (auto-detected)
│   ├── background.ts        # Background service worker
│   ├── popup/               # Popup UI
│   │   ├── index.html
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── options/             # Options page
│   │   ├── index.html
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── content/             # Content scripts
│   │   └── index.ts
│   └── sidepanel/           # Side panel (optional)
│       ├── index.html
│       └── main.tsx
├── components/              # Shared React/Vue components
├── assets/                  # Assets imported in code
├── utils/                   # Utility functions (auto-imported)
├── wxt.config.ts            # WXT configuration
├── package.json
└── .output/                 # Build output (gitignore)
    ├── chrome-mv3/
    └── firefox-mv2/
```

### Key Conventions
```
entrypoints/
├── popup.html              → Popup entry point
├── popup/index.html        → Popup entry point (preferred structure)
├── background.ts           → Background service worker
├── content.ts              → Content script
├── content/                → Content script with multiple files
│   └── index.ts
├── options.html            → Options page
├── sidepanel.html          → Side panel
└── [name].content.ts       → Additional content scripts
```

---

## WXT Configuration

### Basic Configuration (`wxt.config.ts`)
```typescript
import { defineConfig } from 'wxt'

export default defineConfig({
  // Extension metadata
  manifest: {
    name: 'My Extension',
    version: '1.0.0',
    permissions: ['storage', 'activeTab'],
    host_permissions: ['https://*.example.com/*']
  },
  
  // Build configuration
  outDir: '.output',
  srcDir: '.',
  publicDir: 'public',
  
  // Development options
  dev: {
    server: {
      port: 3000
    }
  },
  
  // Browser targets
  browser: 'chrome', // or 'firefox', 'edge', 'safari'
  
  // Module system
  modules: ['@wxt-dev/auto-icons'],
  
  // Vite configuration
  vite: () => ({
    plugins: [],
    build: {
      target: 'esnext'
    }
  })
})
```

### Advanced Configuration
```typescript
import { defineConfig } from 'wxt'
import react from '@vitejs/plugin-react'
import UnoCSS from 'unocss/vite'

export default defineConfig({
  manifest: {
    name: '__MSG_extName__',
    description: '__MSG_extDescription__',
    default_locale: 'en',
    permissions: ['storage', 'tabs', 'scripting'],
    host_permissions: ['<all_urls>'],
    action: {
      default_title: 'My Extension'
    },
    content_scripts: [
      {
        matches: ['https://*.example.com/*'],
        js: ['content-scripts/example.js']
      }
    ]
  },
  
  modules: [
    '@wxt-dev/auto-icons',
    '@wxt-dev/i18n',
    '@wxt-dev/storage'
  ],
  
  vite: () => ({
    plugins: [
      react(),
      UnoCSS()
    ]
  }),
  
  // Import aliases
  alias: {
    '@': './src',
    '~': './components'
  }
})
```

---

## Extension Entry Points

### 1. Popup UI

**entrypoints/popup/index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Extension Popup</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="./main.tsx"></script>
</body>
</html>
```

**entrypoints/popup/main.tsx** (React example)
```typescript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './style.css'

ReactDOM.createRoot(document.getElementById('app')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

**entrypoints/popup/App.tsx**
```typescript
import { useState } from 'react'
import { browser } from 'wxt/browser'
import { storage } from 'wxt/storage'

function App() {
  const [count, setCount] = useState(0)
  
  const handleClick = async () => {
    setCount(count + 1)
    await storage.setItem('local:count', count + 1)
  }
  
  return (
    <div className="popup-container">
      <h1>My Extension</h1>
      <button onClick={handleClick}>
        Count: {count}
      </button>
    </div>
  )
}

export default App
```

### 2. Background Service Worker

**entrypoints/background.ts**
```typescript
import { browser } from 'wxt/browser'

export default defineBackground(() => {
  console.log('Background service worker started')
  
  // Listen for extension install
  browser.runtime.onInstalled.addListener((details) => {
    if (details.reason === 'install') {
      console.log('Extension installed')
      // Open welcome page
      browser.tabs.create({ url: '/welcome.html' })
    }
  })
  
  // Listen for messages
  browser.runtime.onMessage.addListener((message, sender, sendResponse) => {
    console.log('Message received:', message)
    
    if (message.type === 'GET_DATA') {
      // Process request
      sendResponse({ data: 'Hello from background!' })
    }
    
    return true // Keep channel open for async response
  })
  
  // Handle alarms
  browser.alarms.create('periodic-task', { periodInMinutes: 5 })
  
  browser.alarms.onAlarm.addListener((alarm) => {
    if (alarm.name === 'periodic-task') {
      console.log('Running periodic task')
    }
  })
})
```

### 3. Content Scripts

**entrypoints/content.ts** (Basic)
```typescript
import { defineContentScript } from 'wxt/sandbox'

export default defineContentScript({
  matches: ['https://*.example.com/*'],
  runAt: 'document_end',
  
  main() {
    console.log('Content script loaded')
    
    // Inject UI or manipulate DOM
    const button = document.createElement('button')
    button.textContent = 'My Extension Button'
    button.onclick = () => {
      console.log('Button clicked!')
    }
    
    document.body.appendChild(button)
  }
})
```

**entrypoints/content/index.tsx** (React UI injection)
```typescript
import { defineContentScript } from 'wxt/sandbox'
import ReactDOM from 'react-dom/client'
import App from './App'

export default defineContentScript({
  matches: ['https://*.example.com/*'],
  cssInjectionMode: 'ui',
  
  async main(ctx) {
    // Create UI container
    const ui = await createShadowRootUi(ctx, {
      name: 'my-extension-overlay',
      position: 'inline',
      anchor: 'body',
      onMount: (container) => {
        // Mount React app
        const root = ReactDOM.createRoot(container)
        root.render(<App />)
        return root
      },
      onRemove: (root) => {
        root?.unmount()
      }
    })
    
    ui.mount()
  }
})
```

**Content Script with UI Component**
```typescript
// entrypoints/content/App.tsx
import { useState } from 'react'

function App() {
  const [isVisible, setIsVisible] = useState(true)
  
  if (!isVisible) return null
  
  return (
    <div style={{
      position: 'fixed',
      top: 20,
      right: 20,
      zIndex: 999999,
      background: 'white',
      padding: '16px',
      borderRadius: '8px',
      boxShadow: '0 2px 10px rgba(0,0,0,0.1)'
    }}>
      <button onClick={() => setIsVisible(false)}>
        Close
      </button>
      <h3>Extension Panel</h3>
      <p>Content goes here</p>
    </div>
  )
}

export default App
```

### 4. Options Page

**entrypoints/options/index.html**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Extension Settings</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="./main.tsx"></script>
</body>
</html>
```

**entrypoints/options/App.tsx**
```typescript
import { useState, useEffect } from 'react'
import { storage } from 'wxt/storage'

function OptionsApp() {
  const [apiKey, setApiKey] = useState('')
  const [theme, setTheme] = useState('light')
  
  useEffect(() => {
    loadSettings()
  }, [])
  
  const loadSettings = async () => {
    const key = await storage.getItem('local:apiKey')
    const savedTheme = await storage.getItem('local:theme')
    
    if (key) setApiKey(key)
    if (savedTheme) setTheme(savedTheme)
  }
  
  const handleSave = async () => {
    await storage.setItem('local:apiKey', apiKey)
    await storage.setItem('local:theme', theme)
    alert('Settings saved!')
  }
  
  return (
    <div style={{ padding: '40px', maxWidth: '600px' }}>
      <h1>Extension Settings</h1>
      
      <div style={{ marginBottom: '20px' }}>
        <label>
          API Key:
          <input
            type="password"
            value={apiKey}
            onChange={(e) => setApiKey(e.target.value)}
            style={{ display: 'block', marginTop: '8px', width: '100%' }}
          />
        </label>
      </div>
      
      <div style={{ marginBottom: '20px' }}>
        <label>
          Theme:
          <select 
            value={theme}
            onChange={(e) => setTheme(e.target.value)}
            style={{ display: 'block', marginTop: '8px' }}
          >
            <option value="light">Light</option>
            <option value="dark">Dark</option>
          </select>
        </label>
      </div>
      
      <button onClick={handleSave}>
        Save Settings
      </button>
    </div>
  )
}

export default OptionsApp
```

### 5. Side Panel (Chrome 114+)

**entrypoints/sidepanel/index.html**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Side Panel</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="./main.tsx"></script>
</body>
</html>
```

**entrypoints/sidepanel/App.tsx**
```typescript
import { useState, useEffect } from 'react'
import { browser } from 'wxt/browser'

function SidePanelApp() {
  const [currentTab, setCurrentTab] = useState<string>('')
  
  useEffect(() => {
    getCurrentTab()
  }, [])
  
  const getCurrentTab = async () => {
    const [tab] = await browser.tabs.query({ active: true, currentWindow: true })
    setCurrentTab(tab.url || '')
  }
  
  return (
    <div style={{ padding: '20px' }}>
      <h2>Side Panel</h2>
      <p>Current URL: {currentTab}</p>
    </div>
  )
}

export default SidePanelApp
```

---

## WXT Browser API & Auto-imports

### Browser API Access
```typescript
// WXT provides typed browser API
import { browser } from 'wxt/browser'

// Always use 'browser', not 'chrome'
browser.runtime.sendMessage({ type: 'hello' })
browser.storage.local.set({ key: 'value' })
browser.tabs.query({ active: true })
```

### Auto-imports (Like Nuxt)
WXT automatically imports:
- Browser APIs
- WXT utilities
- Your own utility functions from `utils/`

**No imports needed:**
```typescript
// These are auto-imported ✅
defineBackground(() => {
  console.log('Background loaded')
})

defineContentScript({
  matches: ['https://*.example.com/*'],
  main() {
    console.log('Content script')
  }
})

// Utils are auto-imported from utils/
// utils/helpers.ts
export function formatDate(date: Date) {
  return date.toISOString()
}

// Can use directly without import ✅
const formattedDate = formatDate(new Date())
```

### Configure Auto-imports
```typescript
// wxt.config.ts
export default defineConfig({
  imports: {
    addons: {
      vueTemplate: true // For Vue
    },
    presets: [
      'vue',
      'vue-router',
      '@vueuse/core'
    ]
  }
})
```

---

## Storage Management

### WXT Storage Module
```typescript
import { storage } from 'wxt/storage'

// Define storage items with types
const count = storage.defineItem<number>('local:count', {
  defaultValue: 0
})

const settings = storage.defineItem<{ theme: string; apiKey: string }>('local:settings', {
  defaultValue: {
    theme: 'light',
    apiKey: ''
  }
})

// Usage
await count.setValue(10)
const currentCount = await count.getValue() // 10

await settings.setValue({ theme: 'dark', apiKey: 'abc123' })
const currentSettings = await settings.getValue()
```

### Storage Types
```typescript
// Local storage (not synced)
storage.defineItem('local:key', { ... })

// Sync storage (synced across devices)
storage.defineItem('sync:key', { ... })

// Session storage (cleared on browser close)
storage.defineItem('session:key', { ... })

// Managed storage (enterprise policies)
storage.defineItem('managed:key', { ... })
```

### Watch Storage Changes
```typescript
import { storage } from 'wxt/storage'

const count = storage.defineItem<number>('local:count')

// Watch for changes
const unwatch = count.watch((newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`)
})

// Stop watching
unwatch()
```

### React Hook for Storage
```typescript
import { useStorage } from 'wxt/storage'

function MyComponent() {
  const [count, setCount] = useStorage('local:count', 0)
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  )
}
```

---

## Messaging System

### Message Passing with WXT

**Define message type:**
```typescript
// utils/messages.ts
export interface GetDataMessage {
  type: 'GET_DATA'
  payload: {
    id: string
  }
}

export interface SetDataMessage {
  type: 'SET_DATA'
  payload: {
    id: string
    value: string
  }
}

export type ExtensionMessage = GetDataMessage | SetDataMessage
```

**Background message handler:**
```typescript
// entrypoints/background.ts
import { browser } from 'wxt/browser'
import type { ExtensionMessage } from '@/utils/messages'

export default defineBackground(() => {
  browser.runtime.onMessage.addListener((
    message: ExtensionMessage,
    sender,
    sendResponse
  ) => {
    if (message.type === 'GET_DATA') {
      const data = fetchData(message.payload.id)
      sendResponse({ data })
    }
    
    if (message.type === 'SET_DATA') {
      saveData(message.payload.id, message.payload.value)
      sendResponse({ success: true })
    }
    
    return true // Keep channel open
  })
})
```

**Send message from content script or popup:**
```typescript
import { browser } from 'wxt/browser'

const sendMessage = async () => {
  const response = await browser.runtime.sendMessage({
    type: 'GET_DATA',
    payload: { id: '123' }
  })
  
  console.log('Response:', response)
}
```

### Port-based Communication (Long-lived)
```typescript
// Content script
const port = browser.runtime.connect({ name: 'my-channel' })

port.onMessage.addListener((message) => {
  console.log('Received:', message)
})

port.postMessage({ type: 'HELLO' })

// Background
browser.runtime.onConnect.addListener((port) => {
  if (port.name === 'my-channel') {
    port.onMessage.addListener((message) => {
      console.log('From content:', message)
      port.postMessage({ type: 'RESPONSE', data: 'hi' })
    })
  }
})
```

---

## Framework Integration

### React Setup
```bash
pnpm create wxt my-extension --template react
```

**With Tailwind CSS:**
```bash
pnpm add -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

**tailwind.config.js**
```javascript
export default {
  content: [
    './entrypoints/**/*.{html,tsx}',
    './components/**/*.tsx'
  ],
  theme: {
    extend: {}
  },
  plugins: []
}
```

**entrypoints/popup/style.css**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Vue 3 Setup
```bash
pnpm create wxt my-extension --template vue
```

**entrypoints/popup/main.ts**
```typescript
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

**entrypoints/popup/App.vue**
```vue
<script setup lang="ts">
import { ref } from 'vue'
import { storage } from 'wxt/storage'

const count = ref(0)

const increment = async () => {
  count.value++
  await storage.setItem('local:count', count.value)
}
</script>

<template>
  <div class="popup">
    <h1>Vue Extension</h1>
    <button @click="increment">
      Count: {{ count }}
    </button>
  </div>
</template>

<style scoped>
.popup {
  padding: 20px;
  min-width: 300px;
}
</style>
```

### Svelte Setup
```bash
pnpm create wxt my-extension --template svelte
```

**entrypoints/popup/App.svelte**
```svelte
<script lang="ts">
  import { writable } from 'svelte/store'
  import { storage } from 'wxt/storage'
  
  let count = writable(0)
  
  async function increment() {
    count.update(n => n + 1)
    await storage.setItem('local:count', $count)
  }
</script>

<div class="popup">
  <h1>Svelte Extension</h1>
  <button on:click={increment}>
    Count: {$count}
  </button>
</div>

<style>
  .popup {
    padding: 20px;
    min-width: 300px;
  }
</style>
```

### SolidJS Setup
```bash
pnpm create wxt my-extension --template solid
```

---

## Inline Configuration

### Content Script Inline Config
```typescript
export default defineContentScript({
  // Required
  matches: ['https://*.example.com/*'],
  
  // Optional
  excludeMatches: ['https://example.com/admin/*'],
  runAt: 'document_end', // 'document_start' | 'document_end' | 'document_idle'
  allFrames: false,
  world: 'ISOLATED', // 'ISOLATED' | 'MAIN'
  cssInjectionMode: 'ui', // 'ui' | 'manual'
  
  main(ctx) {
    console.log('Content script loaded')
  }
})
```

### Background Inline Config
```typescript
export default defineBackground({
  type: 'module', // or 'classic'
  persistent: false, // MV2 only
  
  main() {
    console.log('Background started')
  }
})
```

---

## Cross-Browser Development

### Build for Multiple Browsers
```typescript
// wxt.config.ts
export default defineConfig({
  // Default browser for dev
  browser: 'chrome',
  
  // Customize manifest per browser
  manifest: ({ browser }) => ({
    name: 'My Extension',
    permissions: browser === 'firefox' 
      ? ['storage', 'tabs']
      : ['storage', 'tabs', 'sidePanel']
  })
})
```

### Build Commands
```bash
# Development
pnpm dev                    # Default browser (chrome)
pnpm dev --browser firefox  # Firefox
pnpm dev --browser edge     # Edge

# Production
pnpm build                  # All browsers
pnpm build:chrome           # Chrome only
pnpm build:firefox          # Firefox only
pnpm build --browser safari # Safari

# Zip for submission
pnpm zip                    # Zip all builds
pnpm zip:chrome             # Zip Chrome build only
```

### Browser-specific Code
```typescript
import { browser } from 'wxt/browser'

// Check current browser
if (import.meta.env.BROWSER === 'chrome') {
  // Chrome-specific code
}

if (import.meta.env.BROWSER === 'firefox') {
  // Firefox-specific code
}

// Runtime check
if (browser.runtime.getURL('').startsWith('chrome-extension://')) {
  console.log('Running in Chrome')
}
```

---

## WXT Modules

### Using Built-in Modules

**@wxt-dev/auto-icons** - Automatically generate all icon sizes
```bash
pnpm add -D @wxt-dev/auto-icons
```

```typescript
// wxt.config.ts
import { defineConfig } from 'wxt'

export default defineConfig({
  modules: ['@wxt-dev/auto-icons']
})
```

Place `icon.png` (512×512) in `public/` and WXT will generate all sizes.

**@wxt-dev/i18n** - Internationalization support
```bash
pnpm add -D @wxt-dev/i18n
```

```typescript
// wxt.config.ts
export default defineConfig({
  modules: ['@wxt-dev/i18n']
})
```

```
public/
└── _locales/
    ├── en/
    │   └── messages.json
    └── vi/
        └── messages.json
```

**@wxt-dev/storage** - Enhanced storage with type safety
```bash
pnpm add @wxt-dev/storage
```

**@wxt-dev/analytics** - Built-in analytics
```bash
pnpm add @wxt-dev/analytics
```

---

## DOM Automation & Web Scraping

### Wait for Elements
```typescript
export default defineContentScript({
  matches: ['https://*.example.com/*'],
  
  main() {
    // Wait for element to appear
    waitForElement('#submit-button').then((button) => {
      button.click()
    })
  }
})

// Utility function (auto-imported from utils/)
export function waitForElement(
  selector: string,
  timeout = 5000
): Promise<Element> {
  return new Promise((resolve, reject) => {
    const element = document.querySelector(selector)
    if (element) return resolve(element)
    
    const observer = new MutationObserver(() => {
      const element = document.querySelector(selector)
      if (element) {
        observer.disconnect()
        resolve(element)
      }
    })
    
    observer.observe(document.body, {
      childList: true,
      subtree: true
    })
    
    setTimeout(() => {
      observer.disconnect()
      reject(new Error(`Element ${selector} not found`))
    }, timeout)
  })
}
```

### Form Automation
```typescript
export default defineContentScript({
  matches: ['https://forms.example.com/*'],
  
  async main() {
    // Wait for form
    const form = await waitForElement('form')
    
    // Fill inputs
    fillFormData({
      email: 'user@example.com',
      password: 'secret123',
      name: 'John Doe'
    })
    
    // Submit
    const submitBtn = await waitForElement('button[type="submit"]')
    submitBtn.click()
  }
})

// utils/formHelpers.ts
export function fillFormData(data: Record<string, string>) {
  Object.entries(data).forEach(([name, value]) => {
    const input = document.querySelector(`[name="${name}"]`) as HTMLInputElement
    if (input) {
      input.value = value
      input.dispatchEvent(new Event('input', { bubbles: true }))
      input.dispatchEvent(new Event('change', { bubbles: true }))
    }
  })
}
```

### Observe DOM Changes
```typescript
export default defineContentScript({
  matches: ['https://*.example.com/*'],
  
  main() {
    // Observe new posts
    const observer = new MutationObserver((mutations) => {
      mutations.forEach((mutation) => {
        mutation.addedNodes.forEach((node) => {
          if (node.nodeType === 1 && node.matches('.post')) {
            console.log('New post detected:', node)
          }
        })
      })
    })
    
    observer.observe(document.body, {
      childList: true,
      subtree: true
    })
  }
})
```

---

## Advanced Patterns

### AI-Powered Extension Example
```typescript
// entrypoints/background.ts
import { defineBackground } from 'wxt/sandbox'

export default defineBackground(() => {
  browser.runtime.onMessage.addListener(async (message, sender, sendResponse) => {
    if (message.type === 'GENERATE_AI_RESPONSE') {
      const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key': import.meta.env.VITE_ANTHROPIC_KEY,
          'anthropic-version': '2023-06-01'
        },
        body: JSON.stringify({
          model: 'claude-sonnet-4-20250514',
          max_tokens: 1024,
          messages: [
            { role: 'user', content: message.prompt }
          ]
        })
      })
      
      const data = await response.json()
      sendResponse({ result: data.content[0].text })
    }
    
    return true
  })
})
```

### Queue-Based Processing
```typescript
// utils/queue.ts
export class TaskQueue {
  private queue: Array<() => Promise<void>> = []
  private processing = false
  private delay = 1000
  
  async add(task: () => Promise<void>) {
    this.queue.push(task)
    if (!this.processing) {
      await this.process()
    }
  }
  
  private async process() {
    this.processing = true
    
    while (this.queue.length > 0) {
      const task = this.queue.shift()
      if (task) {
        await task()
        await new Promise(resolve => setTimeout(resolve, this.delay))
      }
    }
    
    this.processing = false
  }
}

// Usage in background
const queue = new TaskQueue()

browser.runtime.onMessage.addListener((message) => {
  if (message.type === 'PROCESS_ITEM') {
    queue.add(async () => {
      await processItem(message.item)
    })
  }
})
```

---

## Environment Variables

**`.env`**
```bash
# Vite environment variables (prefixed with VITE_)
VITE_API_URL=https://api.example.com
VITE_ANTHROPIC_KEY=sk-ant-xxx
```

**Usage:**
```typescript
// Access in any file
const apiUrl = import.meta.env.VITE_API_URL
const apiKey = import.meta.env.VITE_ANTHROPIC_KEY

// Type-safe access
interface ImportMetaEnv {
  readonly VITE_API_URL: string
  readonly VITE_ANTHROPIC_KEY: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

---

## Build & Deployment

### Production Build
```bash
# Build for all browsers
pnpm build

# Build for specific browser
pnpm build --browser chrome
pnpm build --browser firefox

# Output structure
.output/
├── chrome-mv3/
│   ├── manifest.json
│   ├── background.js
│   └── popup.html
└── firefox-mv2/
    ├── manifest.json
    └── ...
```

### Create Zip for Submission
```bash
# Zip all builds
pnpm zip

# Zip specific browser
pnpm zip:chrome
pnpm zip:firefox

# Output
.output/
├── my-extension-1.0.0-chrome.zip
└── my-extension-1.0.0-firefox.zip
```

### Automated Publishing
```bash
# Install publishing tool
pnpm add -D wxt-publish

# Configure
# wxt.config.ts
export default defineConfig({
  zip: {
    artifactTemplate: '{{name}}-{{version}}-{{browser}}.zip'
  }
})
```

**GitHub Actions CI/CD:**
```yaml
# .github/workflows/release.yml
name: Release Extension

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm build
      - run: pnpm zip
      
      - uses: actions/upload-artifact@v3
        with:
          name: extension-builds
          path: .output/*.zip
```

---

## Debugging

### Debug Different Contexts

**Popup:**
1. Right-click extension icon
2. Select "Inspect popup"
3. DevTools opens for popup

**Background:**
1. Go to `chrome://extensions/`
2. Click "Inspect views: service worker"
3. DevTools opens for background

**Content Scripts:**
1. Open DevTools on the webpage
2. Content script logs appear in page console
3. Use `debugger` statements

**Options Page:**
1. Right-click on options page
2. Select "Inspect"

### Debug Tips
```typescript
// Use descriptive console logs
console.log('[Background] Message received:', message)
console.log('[Content] Button clicked')
console.log('[Popup] Count updated:', count)

// Use debugger statements
export default defineContentScript({
  main() {
    debugger // Pauses execution when DevTools open
    console.log('After debugger')
  }
})

// Log errors with context
try {
  await riskyOperation()
} catch (error) {
  console.error('[Background] Failed to process:', error)
}
```

---

## Common Issues & Solutions

### Issue: Content script not injecting
```typescript
// ❌ Wrong
export default defineContentScript({
  matches: ['example.com/*'], // Missing protocol!
  main() {}
})

// ✅ Correct
export default defineContentScript({
  matches: ['https://example.com/*'], // Include protocol
  main() {}
})
```

### Issue: HMR not working
```bash
# Make sure dev server is running
pnpm dev

# Check browser console for errors
# Reload extension manually if needed
```

### Issue: Storage not persisting
```typescript
// ❌ Wrong
storage.setItem('count', 10) // Missing storage type prefix!

// ✅ Correct
storage.setItem('local:count', 10)
```

### Issue: Messages not working
```typescript
// ❌ Forgot to return true
browser.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  processAsync().then(sendResponse)
  // Missing return true!
})

// ✅ Correct
browser.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  processAsync().then(sendResponse)
  return true // Keep channel open for async
})
```

---

## Best Practices

### 1. Type Safety
```typescript
// Define message types
type MessageType = 
  | { type: 'GET_DATA'; id: string }
  | { type: 'SET_DATA'; id: string; value: string }

// Use in handlers
browser.runtime.onMessage.addListener((msg: MessageType) => {
  if (msg.type === 'GET_DATA') {
    // TypeScript knows msg.id exists
  }
})
```

### 2. Error Handling
```typescript
export default defineBackground(() => {
  browser.runtime.onMessage.addListener(async (message) => {
    try {
      const result = await processMessage(message)
      return { success: true, data: result }
    } catch (error) {
      console.error('Message processing failed:', error)
      return { success: false, error: error.message }
    }
  })
})
```

### 3. Clean Up Resources
```typescript
export default defineContentScript({
  main(ctx) {
    const observer = new MutationObserver(() => {})
    observer.observe(document.body, { childList: true })
    
    // Clean up when content script is removed
    ctx.onInvalidated(() => {
      observer.disconnect()
    })
  }
})
```

### 4. Organize Code
```
entrypoints/
├── background/
│   ├── index.ts
│   ├── messageHandlers.ts
│   └── alarms.ts
├── content/
│   ├── index.ts
│   ├── domHelpers.ts
│   └── uiInjector.ts
└── popup/
    ├── App.tsx
    ├── components/
    └── hooks/
```

---

## Testing

### Unit Testing with Vitest
```bash
pnpm add -D vitest @vitest/ui
```

**vitest.config.ts**
```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    environment: 'happy-dom',
    globals: true
  }
})
```

**Example test:**
```typescript
// utils/helpers.test.ts
import { describe, it, expect } from 'vitest'
import { formatDate } from './helpers'

describe('formatDate', () => {
  it('formats date correctly', () => {
    const date = new Date('2024-01-01')
    expect(formatDate(date)).toBe('2024-01-01')
  })
})
```

---

## Output Requirements

When building WXT extensions, Claude should:

1. **Use WXT conventions** - File-based entry points, auto-imports
2. **Import from `wxt/browser`** - Not `chrome` or raw browser APIs
3. **Use `wxt/storage`** - For typed storage with prefixes
4. **Include TypeScript types** - Full type safety
5. **Use `defineContentScript` / `defineBackground`** - WXT helpers
6. **Support multiple browsers** - Chrome, Firefox, Edge
7. **Follow auto-import patterns** - No unnecessary imports
8. **Include inline configuration** - matches, runAt, etc.
9. **Use Vite environment variables** - VITE_ prefix
10. **Provide complete, working code** - Not snippets

---

## Quick Reference

**Start project:**
```bash
pnpm create wxt my-extension --template react
cd my-extension
pnpm dev
```

**Project structure:**
```
entrypoints/
├── background.ts      → Background service worker
├── popup/            → Popup UI
├── content.ts        → Content script
└── options/          → Settings page
```

**Key imports:**
```typescript
import { browser } from 'wxt/browser'
import { storage } from 'wxt/storage'
import { defineContentScript, defineBackground } from 'wxt/sandbox'
```

**Build commands:**
```bash
pnpm dev              # Development
pnpm build            # Production
pnpm zip              # Create submission zips
```

**Load in browser:**
1. `chrome://extensions/`
2. Enable Developer mode
3. Load unpacked → `.output/chrome-mv3/`

---

## Example: Complete Extension

**File structure:**
```
my-extension/
├── entrypoints/
│   ├── background.ts
│   ├── content.ts
│   └── popup/
│       ├── index.html
│       ├── main.tsx
│       └── App.tsx
├── utils/
│   └── storage.ts
├── wxt.config.ts
└── package.json
```

**wxt.config.ts:**
```typescript
import { defineConfig } from 'wxt'

export default defineConfig({
  manifest: {
    name: 'My Extension',
    permissions: ['storage', 'activeTab']
  },
  modules: ['@wxt-dev/auto-icons']
})
```

**entrypoints/background.ts:**
```typescript
export default defineBackground(() => {
  console.log('Extension loaded')
})
```

**entrypoints/content.ts:**
```typescript
export default defineContentScript({
  matches: ['https://*.example.com/*'],
  main() {
    console.log('Content script active')
  }
})
```

**entrypoints/popup/App.tsx:**
```typescript
import { useState } from 'react'
import { storage } from 'wxt/storage'

export default function App() {
  const [count, setCount] = useState(0)
  
  const increment = async () => {
    const newCount = count + 1
    setCount(newCount)
    await storage.setItem('local:count', newCount)
  }
  
  return (
    <div style={{ padding: 20, minWidth: 300 }}>
      <h1>WXT Extension</h1>
      <button onClick={increment}>Count: {count}</button>
    </div>
  )
}
```

**Run:**
```bash
pnpm dev
```

Extension loads automatically in Chrome! 🎉

---

This skill provides everything needed to build production-ready Chrome Extensions using WXT framework with modern tooling, type safety, and excellent developer experience.
