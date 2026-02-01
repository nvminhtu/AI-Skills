# Chrome Extension Maintenance & CI/CD Skill

## Skill Identity
**Name:** Chrome Extension Maintenance & CI/CD  
**Version:** 1.0.0  
**Purpose:** Automate building, testing, versioning, deployment, and maintenance of Chrome Extensions at scale with professional DevOps practices

## When to Use This Skill
Trigger this skill when the user requests:
- Setting up CI/CD for Chrome extensions
- Automating extension builds and releases
- Version management and changelog generation
- Automated testing for extensions
- Monitoring extension health and errors
- Auto-update strategies
- Multi-browser deployment automation
- Chrome Web Store submission automation
- Extension maintenance workflows
- Keywords: "CI/CD extension", "automate extension release", "github actions extension", "version management", "auto deploy chrome extension", "extension monitoring", "automated testing", "web store automation"

---

## Core Philosophy

### The Extension DevOps Pyramid
```
Level 5: Automated Publishing      ← Full automation
Level 4: Automated Testing          
Level 3: Automated Building         
Level 2: Automated Versioning       
Level 1: Version Control            ← Start here
```

### Benefits of Automation
```
Manual Release (2-4 hours):
1. Update version in manifest.json
2. Update changelog
3. Build extension manually
4. Test in browser
5. Zip files
6. Upload to Chrome Web Store
7. Upload to Firefox Add-ons
8. Update documentation
9. Create GitHub release
10. Notify users

Automated Release (5 minutes):
1. git tag v1.2.3
2. git push --tags
3. Everything else happens automatically ✨
```

---

## 1. Version Control Setup

### Git Repository Structure
```
my-extension/
├── .github/
│   ├── workflows/          # GitHub Actions
│   │   ├── build.yml
│   │   ├── release.yml
│   │   └── test.yml
│   └── ISSUE_TEMPLATE/
├── src/                    # Source code
├── tests/                  # Test files
├── .env.example            # Environment template
├── .gitignore
├── CHANGELOG.md            # Auto-generated
├── package.json
└── README.md
```

### Essential .gitignore
```gitignore
# Build outputs
.output/
dist/
build/
*.zip

# Dependencies
node_modules/
.pnpm-store/

# Environment
.env
.env.local
*.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Extension specific
web-ext-artifacts/
*.crx
*.xpi

# Secrets
*.pem
secrets/
```

### Branch Strategy
```
main (production)
  ↑
develop (staging)
  ↑
feature/* (development)
hotfix/* (emergency fixes)
release/* (release preparation)
```

### Commit Convention (Conventional Commits)
```bash
# Format
<type>(<scope>): <subject>

# Types
feat: New feature
fix: Bug fix
docs: Documentation
style: Formatting
refactor: Code restructuring
test: Tests
chore: Maintenance
perf: Performance

# Examples
feat(popup): add dark mode toggle
fix(content): resolve memory leak on YouTube
docs: update installation guide
chore: upgrade dependencies
```

---

## 2. Automated Versioning

### Semantic Versioning Strategy
```
Version Format: MAJOR.MINOR.PATCH

MAJOR: Breaking changes (1.0.0 → 2.0.0)
MINOR: New features (1.0.0 → 1.1.0)
PATCH: Bug fixes (1.0.0 → 1.0.1)

Pre-release:
1.0.0-alpha.1
1.0.0-beta.1
1.0.0-rc.1
```

### Automated Version Bumping

**Using npm version:**
```bash
# Bump patch version (1.0.0 → 1.0.1)
npm version patch

# Bump minor version (1.0.0 → 1.1.0)
npm version minor

# Bump major version (1.0.0 → 2.0.0)
npm version major

# Pre-release
npm version prerelease --preid=beta
```

**Using standard-version:**
```bash
# Install
pnpm add -D standard-version

# package.json
{
  "scripts": {
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major",
    "release:patch": "standard-version --release-as patch"
  }
}

# Usage
pnpm release          # Auto-detect version bump
pnpm release:minor    # Force minor bump
```

**Configuration (.versionrc.json):**
```json
{
  "types": [
    {"type": "feat", "section": "✨ Features"},
    {"type": "fix", "section": "🐛 Bug Fixes"},
    {"type": "perf", "section": "⚡ Performance"},
    {"type": "docs", "section": "📝 Documentation"},
    {"type": "style", "hidden": true},
    {"type": "refactor", "hidden": true},
    {"type": "test", "hidden": true},
    {"type": "chore", "hidden": true}
  ],
  "skip": {
    "changelog": false,
    "commit": false,
    "tag": false
  },
  "scripts": {
    "postchangelog": "node scripts/update-manifest-version.js"
  }
}
```

**Sync manifest.json version:**
```javascript
// scripts/update-manifest-version.js
const fs = require('fs')
const packageJson = require('../package.json')

// Update manifest.json
const manifestPath = './src/manifest.json'
const manifest = JSON.parse(fs.readFileSync(manifestPath, 'utf8'))
manifest.version = packageJson.version
fs.writeFileSync(manifestPath, JSON.stringify(manifest, null, 2))

// For WXT - update wxt.config.ts
const wxtConfigPath = './wxt.config.ts'
let wxtConfig = fs.readFileSync(wxtConfigPath, 'utf8')
wxtConfig = wxtConfig.replace(
  /version:\s*['"][\d.]+['"]/,
  `version: '${packageJson.version}'`
)
fs.writeFileSync(wxtConfigPath, wxtConfig)

console.log(`✅ Updated version to ${packageJson.version}`)
```

---

## 3. Automated Changelog Generation

### Using Conventional Commits

**Auto-generated CHANGELOG.md:**
```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [1.2.0] - 2024-02-01

### ✨ Features
- **popup**: add dark mode toggle ([#45](link))
- **content**: support YouTube Shorts ([#43](link))

### 🐛 Bug Fixes
- **background**: fix memory leak on long sessions ([#44](link))
- **popup**: correct alignment on small screens

### ⚡ Performance
- optimize content script loading time by 40%

## [1.1.0] - 2024-01-15

### ✨ Features
- **export**: add PDF export functionality

### 🐛 Bug Fixes
- **storage**: prevent data loss on browser crash
```

**Custom Changelog Generator:**
```javascript
// scripts/generate-changelog.js
const { execSync } = require('child_process')
const fs = require('fs')

const getCommitsSinceLastTag = () => {
  try {
    const lastTag = execSync('git describe --tags --abbrev=0', { encoding: 'utf8' }).trim()
    const commits = execSync(`git log ${lastTag}..HEAD --pretty=format:"%H|%s|%an|%ad" --date=short`, { encoding: 'utf8' })
    return commits.split('\n').filter(Boolean).map(commit => {
      const [hash, message, author, date] = commit.split('|')
      return { hash, message, author, date }
    })
  } catch {
    // No tags yet
    return []
  }
}

const categorizeCommits = (commits) => {
  const categories = {
    features: [],
    fixes: [],
    performance: [],
    docs: [],
    others: []
  }
  
  commits.forEach(commit => {
    if (commit.message.startsWith('feat')) {
      categories.features.push(commit)
    } else if (commit.message.startsWith('fix')) {
      categories.fixes.push(commit)
    } else if (commit.message.startsWith('perf')) {
      categories.performance.push(commit)
    } else if (commit.message.startsWith('docs')) {
      categories.docs.push(commit)
    } else {
      categories.others.push(commit)
    }
  })
  
  return categories
}

const generateMarkdown = (version, categories) => {
  let markdown = `## [${version}] - ${new Date().toISOString().split('T')[0]}\n\n`
  
  if (categories.features.length > 0) {
    markdown += '### ✨ Features\n'
    categories.features.forEach(c => {
      markdown += `- ${c.message.replace(/^feat(\([^)]+\))?:\s*/, '')} (${c.hash.substring(0, 7)})\n`
    })
    markdown += '\n'
  }
  
  if (categories.fixes.length > 0) {
    markdown += '### 🐛 Bug Fixes\n'
    categories.fixes.forEach(c => {
      markdown += `- ${c.message.replace(/^fix(\([^)]+\))?:\s*/, '')} (${c.hash.substring(0, 7)})\n`
    })
    markdown += '\n'
  }
  
  if (categories.performance.length > 0) {
    markdown += '### ⚡ Performance\n'
    categories.performance.forEach(c => {
      markdown += `- ${c.message.replace(/^perf(\([^)]+\))?:\s*/, '')} (${c.hash.substring(0, 7)})\n`
    })
    markdown += '\n'
  }
  
  return markdown
}

// Main
const commits = getCommitsSinceLastTag()
const categories = categorizeCommits(commits)
const version = require('../package.json').version
const newChangelog = generateMarkdown(version, categories)

const changelogPath = './CHANGELOG.md'
const existingChangelog = fs.existsSync(changelogPath) 
  ? fs.readFileSync(changelogPath, 'utf8')
  : '# Changelog\n\n'

const updatedChangelog = existingChangelog.replace(
  '# Changelog\n\n',
  `# Changelog\n\n${newChangelog}`
)

fs.writeFileSync(changelogPath, updatedChangelog)
console.log('✅ Changelog updated')
```

---

## 4. Automated Building

### GitHub Actions - Build Workflow

**.github/workflows/build.yml**
```yaml
name: Build Extension

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        browser: [chrome, firefox, edge]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build extension for ${{ matrix.browser }}
        run: pnpm build --browser ${{ matrix.browser }}
        env:
          VITE_API_URL: ${{ secrets.API_URL }}
          VITE_API_KEY: ${{ secrets.API_KEY }}
      
      - name: Create zip
        run: pnpm zip:${{ matrix.browser }}
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v3
        with:
          name: extension-${{ matrix.browser }}
          path: .output/*-${{ matrix.browser }}.zip
          retention-days: 30
      
      - name: Upload source maps (for error tracking)
        uses: actions/upload-artifact@v3
        with:
          name: sourcemaps-${{ matrix.browser }}
          path: .output/${{ matrix.browser }}-mv3/**/*.map
          retention-days: 90

  build-success:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Build success notification
        run: echo "✅ All builds completed successfully"
```

### Multi-stage Build with Caching

**.github/workflows/optimized-build.yml**
```yaml
name: Optimized Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      # Cache node_modules
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.pnpm-store
          key: ${{ runner.os }}-pnpm-${{ hashFiles('pnpm-lock.yaml') }}
          restore-keys: |
            ${{ runner.os }}-pnpm-
      
      # Cache build output
      - name: Cache build
        uses: actions/cache@v3
        with:
          path: .output
          key: ${{ runner.os }}-build-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-build-
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build all browsers
        run: pnpm build
        env:
          NODE_ENV: production
          VITE_API_URL: ${{ secrets.PROD_API_URL }}
      
      - name: Create zips
        run: pnpm zip
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: extension-builds
          path: .output/*.zip
```

---

## 5. Automated Testing

### Test Structure
```
tests/
├── unit/               # Unit tests
│   ├── utils.test.ts
│   └── storage.test.ts
├── integration/        # Integration tests
│   ├── messaging.test.ts
│   └── api.test.ts
├── e2e/               # End-to-end tests
│   ├── popup.test.ts
│   ├── content.test.ts
│   └── workflow.test.ts
└── setup/
    └── test-utils.ts
```

### Unit Testing with Vitest

**vitest.config.ts**
```typescript
import { defineConfig } from 'vitest/config'
import path from 'path'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup/test-utils.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.test.ts'
      ]
    }
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  }
})
```

**Example Unit Tests:**
```typescript
// tests/unit/storage.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { storage } from '@/utils/storage'

// Mock browser API
global.chrome = {
  storage: {
    local: {
      get: vi.fn(),
      set: vi.fn(),
      remove: vi.fn()
    }
  }
} as any

describe('Storage Utils', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })
  
  it('should save data to storage', async () => {
    const data = { count: 5 }
    await storage.set('count', data)
    
    expect(chrome.storage.local.set).toHaveBeenCalledWith(
      { count: data },
      expect.any(Function)
    )
  })
  
  it('should retrieve data from storage', async () => {
    const mockData = { count: { count: 5 } }
    chrome.storage.local.get.mockImplementation((keys, callback) => {
      callback(mockData)
    })
    
    const result = await storage.get('count')
    expect(result).toEqual({ count: 5 })
  })
})
```

### E2E Testing with Playwright

**Install:**
```bash
pnpm add -D @playwright/test
npx playwright install chromium
```

**playwright.config.ts**
```typescript
import { defineConfig } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  timeout: 30000,
  use: {
    headless: false, // See browser during tests
    viewport: { width: 1280, height: 720 },
    trace: 'on-first-retry'
  },
  projects: [
    {
      name: 'chromium',
      use: { 
        browserName: 'chromium',
        launchOptions: {
          args: [
            '--disable-extensions-except=./.output/chrome-mv3',
            '--load-extension=./.output/chrome-mv3'
          ]
        }
      }
    }
  ]
})
```

**E2E Test Example:**
```typescript
// tests/e2e/popup.test.ts
import { test, expect } from '@playwright/test'

test.describe('Extension Popup', () => {
  test('should open popup and display content', async ({ page, context }) => {
    // Load extension
    const extensionId = 'YOUR_EXTENSION_ID' // Get from chrome://extensions
    
    // Open popup
    await page.goto(`chrome-extension://${extensionId}/popup.html`)
    
    // Test popup content
    await expect(page.locator('h1')).toHaveText('My Extension')
    
    // Test button click
    await page.click('button')
    await expect(page.locator('.count')).toHaveText('1')
  })
  
  test('should save settings', async ({ page }) => {
    const extensionId = 'YOUR_EXTENSION_ID'
    await page.goto(`chrome-extension://${extensionId}/options.html`)
    
    // Fill settings
    await page.fill('input[name="apiKey"]', 'test-key-123')
    await page.click('button:text("Save")')
    
    // Verify saved
    await expect(page.locator('.success-message')).toBeVisible()
  })
})
```

### GitHub Actions - Test Workflow

**.github/workflows/test.yml**
```yaml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm test:unit
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          flags: unittests

  e2e-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm build
      
      - name: Install Playwright
        run: npx playwright install --with-deps chromium
      
      - run: pnpm test:e2e
      
      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/

  lint:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm type-check
```

**package.json scripts:**
```json
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:watch": "vitest watch",
    "lint": "eslint . --ext .ts,.tsx",
    "type-check": "tsc --noEmit"
  }
}
```

---

## 6. Automated Release & Publishing

### GitHub Actions - Release Workflow

**.github/workflows/release.yml**
```yaml
name: Release Extension

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  create-release:
    runs-on: ubuntu-latest
    outputs:
      upload_url: ${{ steps.create_release.outputs.upload_url }}
      version: ${{ steps.version.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Get version
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
      
      - name: Generate release notes
        id: release_notes
        run: |
          # Extract changelog for this version
          VERSION=${{ steps.version.outputs.version }}
          NOTES=$(sed -n "/## \[$VERSION\]/,/## \[/p" CHANGELOG.md | sed '$d')
          echo "notes<<EOF" >> $GITHUB_OUTPUT
          echo "$NOTES" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
      
      - name: Create GitHub Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ steps.version.outputs.version }}
          body: ${{ steps.release_notes.outputs.notes }}
          draft: false
          prerelease: false

  build-and-upload:
    needs: create-release
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        browser: [chrome, firefox, edge]
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build extension
        run: pnpm build --browser ${{ matrix.browser }}
        env:
          NODE_ENV: production
          VITE_API_URL: ${{ secrets.PROD_API_URL }}
          VITE_API_KEY: ${{ secrets.PROD_API_KEY }}
      
      - name: Create zip
        run: pnpm zip:${{ matrix.browser }}
      
      - name: Upload to GitHub Release
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ needs.create-release.outputs.upload_url }}
          asset_path: ./.output/my-extension-${{ needs.create-release.outputs.version }}-${{ matrix.browser }}.zip
          asset_name: my-extension-${{ needs.create-release.outputs.version }}-${{ matrix.browser }}.zip
          asset_content_type: application/zip

  publish-chrome:
    needs: [create-release, build-and-upload]
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      - run: pnpm build --browser chrome
      - run: pnpm zip:chrome
      
      - name: Publish to Chrome Web Store
        uses: mnao305/chrome-extension-upload@v4.0.1
        with:
          extension-id: ${{ secrets.CHROME_EXTENSION_ID }}
          client-id: ${{ secrets.CHROME_CLIENT_ID }}
          client-secret: ${{ secrets.CHROME_CLIENT_SECRET }}
          refresh-token: ${{ secrets.CHROME_REFRESH_TOKEN }}
          file-path: ./.output/my-extension-${{ needs.create-release.outputs.version }}-chrome.zip
          publish: true

  publish-firefox:
    needs: [create-release, build-and-upload]
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      - run: pnpm build --browser firefox
      - run: pnpm zip:firefox
      
      - name: Publish to Firefox Add-ons
        uses: wdzeng/edge-addon@v1
        with:
          product-id: ${{ secrets.FIREFOX_EXTENSION_ID }}
          zip-path: ./.output/my-extension-${{ needs.create-release.outputs.version }}-firefox.zip
          client-id: ${{ secrets.FIREFOX_CLIENT_ID }}
          client-secret: ${{ secrets.FIREFOX_CLIENT_SECRET }}

  notify:
    needs: [publish-chrome, publish-firefox]
    runs-on: ubuntu-latest
    
    steps:
      - name: Notify success
        run: echo "🎉 Extension published successfully!"
      
      # Optional: Send notification to Slack/Discord
      - name: Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'New extension version ${{ needs.create-release.outputs.version }} published!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

### Chrome Web Store Credentials Setup

**Get Chrome Web Store API credentials:**
1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create new project
3. Enable Chrome Web Store API
4. Create OAuth 2.0 credentials
5. Get refresh token:

```bash
# Install chrome-webstore-upload-cli
npm install -g chrome-webstore-upload-cli

# Get refresh token
chrome-webstore-upload-cli get-token \
  --client-id YOUR_CLIENT_ID \
  --client-secret YOUR_CLIENT_SECRET

# Save to GitHub Secrets:
# CHROME_CLIENT_ID
# CHROME_CLIENT_SECRET
# CHROME_REFRESH_TOKEN
# CHROME_EXTENSION_ID (from Chrome Web Store)
```

### Firefox Add-ons Credentials Setup

**Get Firefox API credentials:**
1. Go to [AMO Developer Hub](https://addons.mozilla.org/developers/)
2. Generate API credentials
3. Save to GitHub Secrets:
   - `FIREFOX_CLIENT_ID`
   - `FIREFOX_CLIENT_SECRET`
   - `FIREFOX_EXTENSION_ID`

---

## 7. Monitoring & Error Tracking

### Sentry Integration for Extensions

**Install Sentry:**
```bash
pnpm add @sentry/browser
```

**Setup Sentry:**
```typescript
// src/utils/sentry.ts
import * as Sentry from '@sentry/browser'

export const initSentry = () => {
  Sentry.init({
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: import.meta.env.MODE,
    release: `my-extension@${chrome.runtime.getManifest().version}`,
    
    // Performance monitoring
    tracesSampleRate: 0.1,
    
    // Filter errors
    beforeSend(event, hint) {
      // Don't send in development
      if (import.meta.env.DEV) return null
      
      // Filter noise
      if (event.exception?.values?.[0]?.value?.includes('Extension context invalidated')) {
        return null
      }
      
      return event
    },
    
    // Add user context
    initialScope: {
      user: {
        id: getUserId(), // Anonymized user ID
      }
    }
  })
}

// Track custom events
export const trackEvent = (event: string, data?: any) => {
  Sentry.captureEvent({
    message: event,
    level: 'info',
    extra: data
  })
}

// Track errors
export const trackError = (error: Error, context?: any) => {
  Sentry.captureException(error, {
    extra: context
  })
}
```

**Initialize in entrypoints:**
```typescript
// entrypoints/background.ts
import { initSentry } from '@/utils/sentry'

export default defineBackground(() => {
  initSentry()
  
  // Wrap in try-catch
  browser.runtime.onMessage.addListener(async (message) => {
    try {
      // Handle message
    } catch (error) {
      trackError(error, { message })
    }
  })
})
```

### Analytics Integration

**Google Analytics 4 for Extensions:**
```typescript
// src/utils/analytics.ts
import { v4 as uuidv4 } from 'uuid'

const GA_MEASUREMENT_ID = import.meta.env.VITE_GA_MEASUREMENT_ID
const GA_API_SECRET = import.meta.env.VITE_GA_API_SECRET

let userId: string

export const initAnalytics = async () => {
  // Get or create user ID
  const stored = await chrome.storage.local.get('analytics_user_id')
  userId = stored.analytics_user_id || uuidv4()
  
  if (!stored.analytics_user_id) {
    await chrome.storage.local.set({ analytics_user_id: userId })
  }
  
  // Track installation
  chrome.runtime.onInstalled.addListener((details) => {
    if (details.reason === 'install') {
      trackEvent('extension_installed')
    } else if (details.reason === 'update') {
      trackEvent('extension_updated', {
        previous_version: details.previousVersion,
        current_version: chrome.runtime.getManifest().version
      })
    }
  })
}

export const trackEvent = async (
  eventName: string,
  params: Record<string, any> = {}
) => {
  if (!GA_MEASUREMENT_ID || !GA_API_SECRET) return
  
  try {
    await fetch(
      `https://www.google-analytics.com/mp/collect?measurement_id=${GA_MEASUREMENT_ID}&api_secret=${GA_API_SECRET}`,
      {
        method: 'POST',
        body: JSON.stringify({
          client_id: userId,
          events: [
            {
              name: eventName,
              params: {
                ...params,
                extension_version: chrome.runtime.getManifest().version,
                timestamp: Date.now()
              }
            }
          ]
        })
      }
    )
  } catch (error) {
    console.error('Analytics error:', error)
  }
}

export const trackPageView = (pageName: string) => {
  trackEvent('page_view', {
    page_title: pageName,
    page_location: window.location.href
  })
}
```

**Usage:**
```typescript
// Track feature usage
trackEvent('feature_used', {
  feature_name: 'export_pdf',
  user_tier: 'premium'
})

// Track errors
trackEvent('error_occurred', {
  error_message: error.message,
  error_stack: error.stack
})

// Track performance
trackEvent('performance_metric', {
  metric_name: 'page_load_time',
  value: loadTime
})
```

### Health Monitoring Dashboard

**Monitor key metrics:**
```typescript
// src/utils/health-monitor.ts
interface HealthMetrics {
  activeUsers: number
  errorRate: number
  crashRate: number
  avgResponseTime: number
  memoryUsage: number
}

export class HealthMonitor {
  private metrics: HealthMetrics = {
    activeUsers: 0,
    errorRate: 0,
    crashRate: 0,
    avgResponseTime: 0,
    memoryUsage: 0
  }
  
  async reportHealth() {
    // Collect metrics
    const metrics = await this.collectMetrics()
    
    // Send to monitoring service
    await fetch(import.meta.env.VITE_HEALTH_ENDPOINT, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        extension_id: chrome.runtime.id,
        version: chrome.runtime.getManifest().version,
        metrics,
        timestamp: Date.now()
      })
    })
  }
  
  private async collectMetrics(): Promise<HealthMetrics> {
    // Get memory usage
    const memory = (performance as any).memory
    const memoryUsage = memory ? memory.usedJSHeapSize / 1048576 : 0
    
    // Get error rate from storage
    const errorLog = await chrome.storage.local.get('error_log')
    const errors = errorLog.error_log || []
    const recentErrors = errors.filter(
      (e: any) => Date.now() - e.timestamp < 3600000 // Last hour
    )
    
    return {
      ...this.metrics,
      memoryUsage,
      errorRate: recentErrors.length
    }
  }
  
  startMonitoring(intervalMs: number = 300000) {
    // Report every 5 minutes
    setInterval(() => this.reportHealth(), intervalMs)
  }
}

// Initialize in background
export default defineBackground(() => {
  const monitor = new HealthMonitor()
  monitor.startMonitoring()
})
```

---

## 8. Automated Update Strategies

### Auto-update Configuration

**manifest.json (automatically handled by Chrome):**
```json
{
  "update_url": "https://clients2.google.com/service/update2/crx",
  "version": "1.2.3"
}
```

Chrome automatically checks for updates every 5 hours.

### Migration Scripts for Updates

**Handle data migrations:**
```typescript
// src/utils/migrations.ts
interface Migration {
  version: string
  migrate: () => Promise<void>
}

const migrations: Migration[] = [
  {
    version: '2.0.0',
    async migrate() {
      // Migrate storage schema
      const oldData = await chrome.storage.local.get('settings')
      const newData = {
        preferences: {
          theme: oldData.settings?.theme || 'light',
          notifications: oldData.settings?.notifications || true
        }
      }
      await chrome.storage.local.set(newData)
      await chrome.storage.local.remove('settings')
      console.log('Migrated to v2.0.0')
    }
  },
  {
    version: '2.1.0',
    async migrate() {
      // Add new default values
      const current = await chrome.storage.local.get('preferences')
      await chrome.storage.local.set({
        preferences: {
          ...current.preferences,
          autoSave: true // New feature
        }
      })
      console.log('Migrated to v2.1.0')
    }
  }
]

export const runMigrations = async () => {
  const { lastVersion } = await chrome.storage.local.get('lastVersion')
  const currentVersion = chrome.runtime.getManifest().version
  
  // Run migrations
  for (const migration of migrations) {
    if (!lastVersion || compareVersions(migration.version, lastVersion) > 0) {
      try {
        await migration.migrate()
      } catch (error) {
        console.error(`Migration ${migration.version} failed:`, error)
        trackError(error, { migration: migration.version })
      }
    }
  }
  
  // Save current version
  await chrome.storage.local.set({ lastVersion: currentVersion })
}

// Run on extension update
chrome.runtime.onInstalled.addListener((details) => {
  if (details.reason === 'update') {
    runMigrations()
  }
})

function compareVersions(v1: string, v2: string): number {
  const parts1 = v1.split('.').map(Number)
  const parts2 = v2.split('.').map(Number)
  
  for (let i = 0; i < 3; i++) {
    if (parts1[i] > parts2[i]) return 1
    if (parts1[i] < parts2[i]) return -1
  }
  return 0
}
```

### Update Notifications

**Notify users of updates:**
```typescript
// entrypoints/background.ts
chrome.runtime.onInstalled.addListener((details) => {
  if (details.reason === 'update') {
    const currentVersion = chrome.runtime.getManifest().version
    
    // Show notification
    chrome.notifications.create({
      type: 'basic',
      iconUrl: '/icon/128.png',
      title: 'Extension Updated!',
      message: `Updated to version ${currentVersion}. Click to see what's new.`,
      priority: 1
    })
    
    // Track update
    trackEvent('extension_updated', {
      from_version: details.previousVersion,
      to_version: currentVersion
    })
  }
})

// Handle notification click
chrome.notifications.onClicked.addListener(() => {
  chrome.tabs.create({
    url: 'https://your-extension.com/changelog'
  })
})
```

---

## 9. Dependency Management

### Automated Dependency Updates

**Renovate Bot Configuration:**

**renovate.json**
```json
{
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true,
      "automergeType": "pr"
    },
    {
      "matchPackagePatterns": ["^@types/"],
      "automerge": true
    },
    {
      "matchUpdateTypes": ["major"],
      "labels": ["major-update"],
      "automerge": false
    }
  ],
  "schedule": ["before 3am on Monday"],
  "timezone": "America/New_York",
  "prConcurrentLimit": 3,
  "prHourlyLimit": 2
}
```

**Dependabot Configuration:**

**.github/dependabot.yml**
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "03:00"
    open-pull-requests-limit: 5
    reviewers:
      - "your-username"
    assignees:
      - "your-username"
    labels:
      - "dependencies"
    versioning-strategy: increase
    
    # Auto-merge minor and patch updates
    pull-request-branch-name:
      separator: "-"
    
    # Group updates
    groups:
      dev-dependencies:
        patterns:
          - "@types/*"
          - "eslint*"
          - "prettier"
          - "vitest"
      production-dependencies:
        patterns:
          - "@sentry/*"
          - "wxt"
```

### Security Scanning

**.github/workflows/security.yml**
```yaml
name: Security Scan

on:
  schedule:
    - cron: '0 0 * * 1' # Weekly on Monday
  push:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - name: Run audit
        run: pnpm audit --audit-level=moderate
      
      - name: Check for vulnerabilities
        run: pnpm audit --json > audit-report.json || true
      
      - name: Upload audit report
        uses: actions/upload-artifact@v3
        with:
          name: audit-report
          path: audit-report.json

  snyk:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
```

---

## 10. Performance Monitoring

### Bundle Size Tracking

**.github/workflows/bundle-size.yml**
```yaml
name: Bundle Size

on:
  pull_request:
    branches: [main]

jobs:
  bundle-size:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm build
      
      - name: Analyze bundle size
        uses: preactjs/compressed-size-action@v2
        with:
          build-script: "build"
          pattern: ".output/**/*.{js,css}"
          exclude: "{**/*.map,**/node_modules/**}"
          compression: "gzip"
```

### Performance Budgets

**vite.config.ts**
```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'date-fns']
        }
      }
    },
    // Warn if chunks exceed limits
    chunkSizeWarningLimit: 500, // KB
  }
})
```

---

## 11. Documentation Automation

### Auto-generate API Documentation

**TypeDoc Configuration:**

**typedoc.json**
```json
{
  "entryPoints": ["src"],
  "out": "docs",
  "plugin": ["typedoc-plugin-markdown"],
  "readme": "README.md",
  "exclude": [
    "**/*.test.ts",
    "**/node_modules/**"
  ],
  "excludePrivate": true,
  "excludeProtected": true
}
```

**Generate docs:**
```bash
pnpm add -D typedoc typedoc-plugin-markdown

# package.json
{
  "scripts": {
    "docs": "typedoc"
  }
}

pnpm docs
```

### Auto-generate README badges

**README.md**
```markdown
# My Extension

[![Version](https://img.shields.io/github/v/release/username/repo)](https://github.com/username/repo/releases)
[![Build](https://img.shields.io/github/actions/workflow/status/username/repo/build.yml)](https://github.com/username/repo/actions)
[![Tests](https://img.shields.io/github/actions/workflow/status/username/repo/test.yml?label=tests)](https://github.com/username/repo/actions)
[![Coverage](https://img.shields.io/codecov/c/github/username/repo)](https://codecov.io/gh/username/repo)
[![License](https://img.shields.io/github/license/username/repo)](LICENSE)
[![Users](https://img.shields.io/chrome-web-store/users/EXTENSION_ID)](https://chrome.google.com/webstore/detail/EXTENSION_ID)
[![Rating](https://img.shields.io/chrome-web-store/rating/EXTENSION_ID)](https://chrome.google.com/webstore/detail/EXTENSION_ID)
```

---

## 12. Rollback Strategy

### Automatic Rollback on Errors

**Monitor error rate after release:**
```typescript
// scripts/monitor-release.js
const SENTRY_API = 'https://sentry.io/api/0/projects/ORG/PROJECT/events/'
const ERROR_THRESHOLD = 10 // errors per minute

const checkErrorRate = async () => {
  const response = await fetch(SENTRY_API, {
    headers: {
      'Authorization': `Bearer ${process.env.SENTRY_AUTH_TOKEN}`
    }
  })
  
  const events = await response.json()
  const recentErrors = events.filter(
    e => Date.now() - new Date(e.dateCreated).getTime() < 60000
  )
  
  if (recentErrors.length > ERROR_THRESHOLD) {
    console.error('🚨 High error rate detected! Rolling back...')
    
    // Trigger rollback
    await rollback()
  }
}

const rollback = async () => {
  // Revert to previous version
  execSync('git revert HEAD')
  execSync('git push')
  
  // Notify team
  await notifySlack('Extension rolled back due to high error rate')
}

// Monitor for 1 hour after release
setInterval(checkErrorRate, 60000)
setTimeout(() => process.exit(0), 3600000)
```

### Manual Rollback Procedure

**scripts/rollback.sh**
```bash
#!/bin/bash

# Rollback to previous version

PREV_VERSION=$1

if [ -z "$PREV_VERSION" ]; then
  echo "Usage: ./rollback.sh <version>"
  exit 1
fi

echo "Rolling back to version $PREV_VERSION"

# Checkout previous version
git checkout "v$PREV_VERSION"

# Build
pnpm install
pnpm build

# Upload to Chrome Web Store
pnpm publish:chrome

echo "✅ Rolled back to $PREV_VERSION"
```

---

## 13. Complete CI/CD Pipeline Example

### Full Pipeline (.github/workflows/pipeline.yml)

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, develop]
    tags:
      - 'v*.*.*'
  pull_request:
    branches: [main]

jobs:
  # Stage 1: Code Quality
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm type-check
      - run: pnpm audit

  # Stage 2: Testing
  test:
    needs: quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm test:unit --coverage
      
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json

  # Stage 3: Build
  build:
    needs: test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chrome, firefox, edge]
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm build --browser ${{ matrix.browser }}
        env:
          VITE_API_URL: ${{ secrets.PROD_API_URL }}
          VITE_SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
      
      - run: pnpm zip:${{ matrix.browser }}
      
      - uses: actions/upload-artifact@v3
        with:
          name: extension-${{ matrix.browser }}
          path: .output/*-${{ matrix.browser }}.zip

  # Stage 4: E2E Tests
  e2e:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install
      
      - uses: actions/download-artifact@v3
        with:
          name: extension-chrome
      
      - run: npx playwright install --with-deps chromium
      - run: pnpm test:e2e

  # Stage 5: Release (only on tags)
  release:
    if: startsWith(github.ref, 'refs/tags/v')
    needs: [build, e2e]
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Get version
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
      
      - uses: actions/download-artifact@v3
      
      - name: Create Release
        uses: ncipollo/release-action@v1
        with:
          artifacts: "extension-*/*.zip"
          generateReleaseNotes: true
          token: ${{ secrets.GITHUB_TOKEN }}

  # Stage 6: Publish (only on tags)
  publish-chrome:
    if: startsWith(github.ref, 'refs/tags/v')
    needs: release
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v3
        with:
          name: extension-chrome
      
      - name: Publish to Chrome Web Store
        uses: mnao305/chrome-extension-upload@v4.0.1
        with:
          extension-id: ${{ secrets.CHROME_EXTENSION_ID }}
          client-id: ${{ secrets.CHROME_CLIENT_ID }}
          client-secret: ${{ secrets.CHROME_CLIENT_SECRET }}
          refresh-token: ${{ secrets.CHROME_REFRESH_TOKEN }}
          file-path: ./extension-chrome/*.zip
          publish: true
      
      - name: Notify success
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -H 'Content-Type: application/json' \
            -d '{"text":"✅ Chrome extension published successfully!"}'

  publish-firefox:
    if: startsWith(github.ref, 'refs/tags/v')
    needs: release
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v3
        with:
          name: extension-firefox
      
      - name: Publish to Firefox Add-ons
        run: |
          # Firefox publishing logic
          echo "Publishing to Firefox..."
```

---

## 14. Best Practices Checklist

### Pre-Release Checklist

```markdown
## Pre-Release Checklist

### Code Quality
- [ ] All tests passing
- [ ] Code coverage > 80%
- [ ] No linting errors
- [ ] TypeScript type checks pass
- [ ] No security vulnerabilities

### Documentation
- [ ] CHANGELOG.md updated
- [ ] README.md updated
- [ ] API docs generated
- [ ] Migration guide written (if breaking changes)

### Testing
- [ ] Manual testing completed
- [ ] Tested on Chrome/Firefox/Edge
- [ ] Tested with different user states
- [ ] Performance regression tests pass

### Build
- [ ] Version bumped correctly
- [ ] manifest.json version matches package.json
- [ ] Build artifacts generated successfully
- [ ] Bundle size within limits

### Monitoring
- [ ] Sentry configured
- [ ] Analytics tracking verified
- [ ] Error logging tested

### Release
- [ ] GitHub release created
- [ ] Assets uploaded
- [ ] Store listings updated
- [ ] Users notified (if major update)
```

---

## Output Requirements

When helping users set up CI/CD for Chrome Extensions, Claude should:

1. **Provide complete workflow files** - Not partial examples
2. **Include all necessary secrets** - Document what needs to be configured
3. **Set up proper versioning** - Semantic versioning with automation
4. **Include testing at all stages** - Unit, integration, E2E
5. **Add monitoring and error tracking** - Sentry, analytics
6. **Automate changelog generation** - From commit messages
7. **Configure multi-browser builds** - Chrome, Firefox, Edge
8. **Set up automated publishing** - To all stores
9. **Include rollback procedures** - For when things go wrong
10. **Provide maintenance scripts** - For common tasks

---

## Quick Start Example

**Complete setup in 10 minutes:**

```bash
# 1. Initialize project
pnpm create wxt my-extension
cd my-extension

# 2. Add CI/CD tools
pnpm add -D standard-version vitest @playwright/test

# 3. Copy workflow files
mkdir -p .github/workflows
# Add build.yml, test.yml, release.yml

# 4. Configure scripts in package.json
{
  "scripts": {
    "release": "standard-version",
    "test:unit": "vitest run --coverage",
    "test:e2e": "playwright test"
  }
}

# 5. Set up GitHub secrets
# Go to Settings > Secrets and add:
# - CHROME_CLIENT_ID
# - CHROME_CLIENT_SECRET
# - CHROME_REFRESH_TOKEN
# - CHROME_EXTENSION_ID
# - SENTRY_DSN
# - SLACK_WEBHOOK

# 6. First release
pnpm release
git push --follow-tags

# ✅ CI/CD is now fully automated!
```

---

This skill covers everything needed to maintain and deploy Chrome Extensions professionally with full automation, monitoring, and best practices.
