# Plasmo Chrome Extension Development Skill

## Skill Identity
**Name:** Plasmo Chrome Extension Development  
**Version:** 1.0.0  
**Purpose:** Build production-ready Chrome Extensions (Manifest V3) using Plasmo framework with React and TypeScript

## When to Use This Skill
Trigger this skill when the user requests:
- Creating a new Chrome extension
- Building browser automation tools
- Injecting UI components into web pages
- Developing popup, options, or content script interfaces
- Setting up extension messaging and storage
- Automating web workflows
- Building AI-powered browser extensions
- Keywords: "chrome extension", "browser extension", "Plasmo", "inject UI", "content script", "popup", "browser automation"

## Core Framework Philosophy
Plasmo is the "Next.js for Chrome Extensions" - it uses:
- **File-based routing**: Directory structure automatically maps to extension components
- **Zero-config approach**: Auto-generates manifest.json
- **React-first**: Built for modern React + TypeScript development
- **Hot reload**: Fast development workflow with instant updates

## Project Structure & Setup

### Initialize New Project
```bash
# Using pnpm (recommended)
pnpm create plasmo

# Using npm
npm create plasmo

# Development
pnpm dev

# Production build
pnpm build
```

### Standard Project Structure
```
my-extension/
├── src/
│   ├── popup.tsx           # Popup UI (automatic)
│   ├── options.tsx         # Options page (automatic)
│   ├── background.ts       # Service worker (automatic)
│   ├── contents/           # Content scripts
│   │   └── example.tsx
│   ├── components/         # Shared React components
│   └── assets/             # Images, icons
├── package.json
└── .plasmo/                # Generated files (gitignore)
```

## Extension Components

### 1. Popup UI (`popup.tsx`)
```typescript
import { useState } from "react"
import "./popup.css"

function IndexPopup() {
  const [data, setData] = useState("")

  return (
    <div style={{ padding: 16, minWidth: 300 }}>
      <h2>Extension Popup</h2>
      <input 
        onChange={(e) => setData(e.target.value)}
        value={data}
      />
      <button onClick={() => console.log(data)}>
        Submit
      </button>
    </div>
  )
}

export default IndexPopup
```

### 2. Content Scripts (`contents/example.tsx`)
```typescript
import type { PlasmoCSConfig } from "plasmo"
import { useState } from "react"

// Configure which pages this content script runs on
export const config: PlasmoCSConfig = {
  matches: ["https://*.example.com/*"],
  all_frames: false
}

// Content script UI component (injected into page)
const Example = () => {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div
      style={{
        position: "fixed",
        top: 20,
        right: 20,
        zIndex: 9999,
        background: "white",
        padding: "12px",
        borderRadius: "8px",
        boxShadow: "0 2px 10px rgba(0,0,0,0.1)"
      }}>
      <button onClick={() => setIsOpen(!isOpen)}>
        Toggle Panel
      </button>
      {isOpen && <div>Content here</div>}
    </div>
  )
}

export default Example
```

### 3. Background Service Worker (`background.ts`)
```typescript
// Background scripts run in service worker context
console.log("Background service worker started")

// Listen for extension install
chrome.runtime.onInstalled.addListener(() => {
  console.log("Extension installed")
})

// Listen for messages
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  console.log("Message received:", request)
  sendResponse({ status: "ok" })
  return true // Required for async response
})

// Handle alarms, periodic tasks
chrome.alarms.create("periodic-task", { periodInMinutes: 5 })

chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === "periodic-task") {
    console.log("Running periodic task")
  }
})
```

### 4. Options Page (`options.tsx`)
```typescript
import { useState, useEffect } from "react"
import { Storage } from "@plasmohq/storage"

const storage = new Storage()

function OptionsIndex() {
  const [apiKey, setApiKey] = useState("")

  useEffect(() => {
    storage.get("apiKey").then(setApiKey)
  }, [])

  const handleSave = async () => {
    await storage.set("apiKey", apiKey)
    alert("Settings saved!")
  }

  return (
    <div style={{ padding: 40 }}>
      <h1>Extension Settings</h1>
      <label>
        API Key:
        <input
          type="password"
          value={apiKey}
          onChange={(e) => setApiKey(e.target.value)}
        />
      </label>
      <button onClick={handleSave}>Save</button>
    </div>
  )
}

export default OptionsIndex
```

## Messaging System

### Plasmo Messaging Architecture
Create message handlers in `background/messages/` directory:

```typescript
// background/messages/ping.ts
import type { PlasmoMessaging } from "@plasmohq/messaging"

const handler: PlasmoMessaging.MessageHandler = async (req, res) => {
  const { name } = req.body
  
  res.send({
    message: `Hello ${name}!`
  })
}

export default handler
```

### Send Messages from Content Script or Popup
```typescript
import { sendToBackground } from "@plasmohq/messaging"

const response = await sendToBackground({
  name: "ping",
  body: {
    name: "World"
  }
})

console.log(response.message) // "Hello World!"
```

### Content Script ↔ Background Communication
```typescript
// In content script
const sendMessageToBackground = async () => {
  const response = await chrome.runtime.sendMessage({
    type: "GET_DATA",
    payload: { id: 123 }
  })
  return response
}

// In background.ts
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === "GET_DATA") {
    // Process data
    sendResponse({ data: "result" })
  }
  return true // IMPORTANT: Required for async
})
```

## Storage Management

### Using Plasmo Storage
```typescript
import { Storage } from "@plasmohq/storage"

const storage = new Storage()

// Save data
await storage.set("user", { name: "John", age: 30 })

// Get data
const user = await storage.get("user")

// Watch for changes
storage.watch({
  "user": (change) => {
    console.log("User changed:", change.newValue)
  }
})

// Use React hook
import { useStorage } from "@plasmohq/storage/hook"

function MyComponent() {
  const [count, setCount] = useStorage("count", 0)
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

### Chrome Storage API Directly
```typescript
// Local storage (not synced)
await chrome.storage.local.set({ key: "value" })
const { key } = await chrome.storage.local.get("key")

// Sync storage (synced across devices)
await chrome.storage.sync.set({ settings: {...} })
```

## DOM Automation & Interaction

### Safe DOM Manipulation
```typescript
// Wait for element to appear
const waitForElement = (selector: string, timeout = 5000): Promise<Element> => {
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

// Usage
const button = await waitForElement("#submit-btn")
button.click()
```

### Form Automation
```typescript
// Fill form fields
const fillForm = (formData: Record<string, string>) => {
  Object.entries(formData).forEach(([name, value]) => {
    const input = document.querySelector(`[name="${name}"]`) as HTMLInputElement
    if (input) {
      input.value = value
      input.dispatchEvent(new Event("input", { bubbles: true }))
      input.dispatchEvent(new Event("change", { bubbles: true }))
    }
  })
}

fillForm({
  email: "user@example.com",
  password: "secret123"
})
```

### Observe Page Changes
```typescript
// Watch for dynamic content
const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    mutation.addedNodes.forEach((node) => {
      if (node.nodeType === 1) { // Element node
        console.log("New element added:", node)
      }
    })
  })
})

observer.observe(document.body, {
  childList: true,
  subtree: true,
  attributes: true
})
```

## Styling Strategies

### 1. CSS Modules (Recommended)
```typescript
// popup.module.css
.container {
  padding: 16px;
  background: #f0f0f0;
}

// popup.tsx
import styles from "./popup.module.css"

function Popup() {
  return <div className={styles.container}>Content</div>
}
```

### 2. Inline Styles (Content Scripts)
```typescript
// Avoid CSS conflicts with host page
const MyOverlay = () => (
  <div style={{
    position: "fixed",
    top: 0,
    right: 0,
    zIndex: 999999,
    background: "white",
    boxShadow: "0 2px 10px rgba(0,0,0,0.2)"
  }}>
    Overlay Content
  </div>
)
```

### 3. Shadow DOM (Isolation)
```typescript
import { useState } from "react"
import type { PlasmoCSConfig } from "plasmo"

export const config: PlasmoCSConfig = {
  matches: ["https://*.example.com/*"]
}

// Plasmo automatically creates Shadow DOM
export const getShadowHostId = () => "my-extension-root"

const ContentScript = () => {
  return (
    <style-wrapper>
      <style>{`
        .my-button {
          background: blue;
          color: white;
        }
      `}</style>
      <button className="my-button">Click Me</button>
    </style-wrapper>
  )
}

export default ContentScript
```

## Permissions & Manifest

### Common Permissions
```typescript
// package.json - Plasmo automatically generates manifest
{
  "manifest": {
    "permissions": [
      "storage",
      "activeTab",
      "scripting",
      "alarms"
    ],
    "host_permissions": [
      "https://*.example.com/*"
    ]
  }
}
```

### Request Optional Permissions
```typescript
const requestPermission = async () => {
  const granted = await chrome.permissions.request({
    permissions: ["tabs"],
    origins: ["https://api.example.com/*"]
  })
  
  if (granted) {
    console.log("Permission granted")
  }
}
```

## Build & Deployment

### Production Build
```bash
# Build for Chrome
pnpm build

# Build for Firefox
pnpm build --target=firefox-mv3

# Build for multiple browsers
pnpm build --target=chrome-mv3,firefox-mv3
```

### Output Structure
```
build/
├── chrome-mv3-prod/
│   ├── manifest.json
│   ├── popup.html
│   ├── background.js
│   └── contents/
```

### Load Extension in Chrome
1. Open `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select `build/chrome-mv3-prod/` folder

## Advanced Patterns

### AI-Powered Extensions
```typescript
// background/messages/ai-process.ts
import type { PlasmoMessaging } from "@plasmohq/messaging"

const handler: PlasmoMessaging.MessageHandler = async (req, res) => {
  const { prompt } = req.body
  
  // Call AI API
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.PLASMO_PUBLIC_ANTHROPIC_KEY
    },
    body: JSON.stringify({
      model: "claude-sonnet-4-20250514",
      max_tokens: 1024,
      messages: [{ role: "user", content: prompt }]
    })
  })
  
  const data = await response.json()
  res.send({ result: data.content[0].text })
}

export default handler
```

### Queue-Based Processing
```typescript
class TaskQueue {
  private queue: Array<() => Promise<void>> = []
  private processing = false
  private delay = 1000 // 1 second between tasks

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

// Usage
const queue = new TaskQueue()
queue.add(async () => {
  console.log("Task 1")
})
```

## Debugging Tips

### Debug Content Scripts
1. Open DevTools on the target webpage
2. Content script logs appear in the page console
3. Use `debugger` statements for breakpoints

### Debug Background Service Worker
1. Go to `chrome://extensions/`
2. Click "Inspect views: service worker"
3. DevTools opens for background context

### Debug Popup
1. Right-click extension icon
2. Select "Inspect popup"
3. DevTools opens for popup context

### Common Issues
```typescript
// Issue: Service worker stops after 30 seconds
// Solution: Keep alive with periodic messages
chrome.runtime.onInstalled.addListener(() => {
  chrome.alarms.create("keep-alive", { periodInMinutes: 1 })
})

// Issue: Content script not injecting
// Solution: Check matches pattern and permissions
export const config: PlasmoCSConfig = {
  matches: ["<all_urls>"], // Or specific pattern
  run_at: "document_end" // Or "document_start", "document_idle"
}

// Issue: CORS errors
// Solution: Make requests from background script
// Use chrome.runtime.sendMessage to proxy requests
```

## Environment Variables
```bash
# .env.local
PLASMO_PUBLIC_API_URL=https://api.example.com
PLASMO_PUBLIC_API_KEY=secret_key_123
```

```typescript
// Access in code
const apiUrl = process.env.PLASMO_PUBLIC_API_URL
```

## Best Practices

1. **Security**
   - Never expose API keys in content scripts
   - Use background scripts for sensitive operations
   - Validate all user inputs
   - Use Content Security Policy

2. **Performance**
   - Minimize content script size
   - Use lazy loading for heavy components
   - Debounce DOM observations
   - Clean up listeners and observers

3. **User Experience**
   - Show loading states
   - Handle errors gracefully
   - Provide clear feedback
   - Respect user privacy

4. **Code Organization**
   - Separate business logic from UI
   - Use TypeScript for type safety
   - Create reusable components
   - Document complex functionality

## Testing Strategy
```typescript
// Example unit test with Jest
import { render, screen } from "@testing-library/react"
import Popup from "../popup"

test("renders popup", () => {
  render(<Popup />)
  expect(screen.getByText("Extension Popup")).toBeInTheDocument()
})
```

## CI/CD Example (GitHub Actions)
```yaml
# .github/workflows/build.yml
name: Build Extension

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm build
      
      - uses: actions/upload-artifact@v3
        with:
          name: extension-build
          path: build/chrome-mv3-prod/
```

## Output Requirements

When building Plasmo extensions, Claude should:
1. **Always create complete, working code** - not snippets
2. **Include proper TypeScript types**
3. **Follow Plasmo file conventions** (popup.tsx, background.ts, etc.)
4. **Handle errors gracefully**
5. **Add comments for complex logic**
6. **Consider security and permissions**
7. **Provide setup instructions**
8. **Include debugging tips when relevant**

## Example Project Templates

### Minimal Popup Extension
```
my-extension/
├── src/
│   └── popup.tsx
├── package.json
└── assets/
    └── icon.png
```

### Content Script + Background
```
my-extension/
├── src/
│   ├── background.ts
│   ├── contents/
│   │   └── main.tsx
│   └── components/
│       └── Overlay.tsx
├── package.json
└── .env.local
```

### Full-Featured Extension
```
my-extension/
├── src/
│   ├── popup.tsx
│   ├── options.tsx
│   ├── background.ts
│   ├── background/messages/
│   │   ├── getData.ts
│   │   └── processAI.ts
│   ├── contents/
│   │   ├── inject-ui.tsx
│   │   └── automation.ts
│   ├── components/
│   │   ├── Panel.tsx
│   │   └── Settings.tsx
│   └── lib/
│       ├── storage.ts
│       └── utils.ts
├── package.json
├── .env.local
└── assets/
```

---

## Quick Reference

**Start new project:** `pnpm create plasmo`  
**Dev mode:** `pnpm dev`  
**Build:** `pnpm build`  
**Load extension:** chrome://extensions → Load unpacked → build/chrome-mv3-prod/

**Key imports:**
- `import { Storage } from "@plasmohq/storage"`
- `import { sendToBackground } from "@plasmohq/messaging"`
- `import type { PlasmoCSConfig } from "plasmo"`

**Common patterns:**
- Popup UI: `popup.tsx`
- Content script: `contents/name.tsx` + export config
- Background: `background.ts` or `background/index.ts`
- Message handler: `background/messages/name.ts`
