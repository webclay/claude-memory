# Chrome Extension Development - Manifest V3 Patterns

**Purpose:** Comprehensive guide for developing Chrome extensions with Manifest V3
**Last Updated:** 2025-01-12
**Manifest Version:** V3 (Required as of January 2024)
**Chrome Version:** 88+

---

## Overview

Chrome extensions enhance browser functionality using web technologies (HTML, CSS, JavaScript). Manifest V3 is the current standard, bringing improved security, privacy, and performance.

**Key Components:**
- Manifest file (configuration)
- Background Service Workers (replaces background pages)
- Content Scripts (inject into web pages)
- Popup HTML (extension icon click)
- Options Page (extension settings)
- Message passing between components
- Chrome APIs (storage, tabs, bookmarks, etc.)

---

## Extension Structure

### Recommended File Structure

```
my-extension/
├── manifest.json              # Required configuration file
├── background/
│   └── service-worker.js      # Background service worker
├── content/
│   ├── content-script.js      # Injected into pages
│   └── content-styles.css     # Styles for injected content
├── popup/
│   ├── popup.html             # Extension popup UI
│   ├── popup.js               # Popup logic
│   └── popup.css              # Popup styles
├── options/
│   ├── options.html           # Settings page
│   ├── options.js             # Settings logic
│   └── options.css            # Settings styles
├── utils/
│   ├── storage.js             # Storage helpers
│   └── api.js                 # API utilities
├── assets/
│   └── icons/
│       ├── icon16.png         # Required
│       ├── icon48.png         # Required
│       ├── icon128.png        # Required
│       └── icon512.png        # For Chrome Web Store
└── lib/                       # Third-party libraries (if any)
```

---

## Manifest.json Configuration

### Basic Manifest V3

```json
{
  "manifest_version": 3,
  "name": "My Awesome Extension",
  "version": "1.0.0",
  "description": "Brief description of what your extension does (132 characters max)",
  "author": "Your Name",

  "icons": {
    "16": "assets/icons/icon16.png",
    "48": "assets/icons/icon48.png",
    "128": "assets/icons/icon128.png"
  },

  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "assets/icons/icon16.png",
      "48": "assets/icons/icon48.png",
      "128": "assets/icons/icon128.png"
    },
    "default_title": "Click to open extension"
  },

  "background": {
    "service_worker": "background/service-worker.js"
  },

  "content_scripts": [
    {
      "matches": ["https://*/*", "http://*/*"],
      "js": ["content/content-script.js"],
      "css": ["content/content-styles.css"],
      "run_at": "document_idle"
    }
  ],

  "permissions": [
    "storage",
    "activeTab"
  ],

  "host_permissions": [
    "https://api.example.com/*"
  ],

  "options_page": "options/options.html",

  "web_accessible_resources": [
    {
      "resources": ["assets/*"],
      "matches": ["<all_urls>"]
    }
  ]
}
```

### Common Permissions

```json
{
  "permissions": [
    "storage",              // Use chrome.storage API
    "activeTab",            // Access current tab (user-initiated)
    "tabs",                 // Access tab information
    "bookmarks",            // Access bookmarks
    "history",              // Access browsing history
    "cookies",              // Access cookies
    "notifications",        // Show notifications
    "contextMenus",         // Add context menu items
    "alarms",               // Schedule code to run
    "scripting",            // Inject scripts programmatically
    "declarativeNetRequest" // Block/modify network requests
  ],

  "host_permissions": [
    "https://api.example.com/*",  // Specific API domain
    "https://*.example.com/*",     // All subdomains
    "<all_urls>"                   // All URLs (use sparingly!)
  ]
}
```

---

## Background Service Worker

### Basic Service Worker

Service workers replace persistent background pages in Manifest V3. They're event-driven and terminate when idle.

**background/service-worker.js:**

```javascript
// Listen for extension installation
chrome.runtime.onInstalled.addListener(({ reason }) => {
  if (reason === 'install') {
    console.log('Extension installed');

    // Set default settings
    chrome.storage.sync.set({
      enabled: true,
      theme: 'light'
    });

    // Open welcome page
    chrome.tabs.create({
      url: 'options/options.html'
    });
  } else if (reason === 'update') {
    console.log('Extension updated');
  }
});

// Listen for messages from popup or content scripts
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  console.log('Received message:', message);

  if (message.action === 'getData') {
    // Fetch data from API
    fetchDataFromAPI()
      .then(data => sendResponse({ success: true, data }))
      .catch(error => sendResponse({ success: false, error: error.message }));

    return true; // Keep channel open for async response
  }

  if (message.action === 'saveData') {
    chrome.storage.local.set({ myData: message.data }, () => {
      sendResponse({ success: true });
    });

    return true;
  }
});

// Listen for tab updates
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete' && tab.url) {
    console.log('Tab loaded:', tab.url);

    // Inject content script programmatically if needed
    if (tab.url.includes('example.com')) {
      chrome.scripting.executeScript({
        target: { tabId: tabId },
        files: ['content/content-script.js']
      });
    }
  }
});

// Scheduled tasks with alarms
chrome.alarms.create('dailyTask', { periodInMinutes: 1440 }); // 24 hours

chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'dailyTask') {
    console.log('Running daily task');
    performDailyTask();
  }
});

// Helper function
async function fetchDataFromAPI() {
  const response = await fetch('https://api.example.com/data', {
    headers: {
      'Content-Type': 'application/json'
    }
  });

  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }

  return response.json();
}

function performDailyTask() {
  // Your scheduled task logic
}
```

### Service Worker Lifecycle

**Important:** Service workers terminate when idle (after ~30 seconds of inactivity).

```javascript
// ✅ CORRECT: Use chrome.storage (persists when worker terminates)
chrome.storage.local.set({ data: 'value' });

// ❌ WRONG: Variables reset when worker terminates
let myData = 'value'; // Will be lost!

// ✅ CORRECT: Use chrome.alarms for scheduled tasks
chrome.alarms.create('reminder', { delayInMinutes: 30 });

// ❌ WRONG: setTimeout won't survive worker termination
setTimeout(() => {}, 30 * 60 * 1000); // Will fail!
```

---

## Content Scripts

Content scripts run in the context of web pages and can read/modify the DOM.

### Basic Content Script

**content/content-script.js:**

```javascript
// Content scripts run on every page matching manifest patterns
console.log('Content script loaded on:', window.location.href);

// Access and modify the page DOM
function highlightKeywords() {
  const keywords = ['important', 'urgent'];

  const walker = document.createTreeWalker(
    document.body,
    NodeFilter.SHOW_TEXT,
    null
  );

  let node;
  while (node = walker.nextNode()) {
    keywords.forEach(keyword => {
      if (node.textContent.includes(keyword)) {
        const span = document.createElement('span');
        span.style.backgroundColor = 'yellow';
        span.textContent = node.textContent;
        node.parentNode.replaceChild(span, node);
      }
    });
  }
}

// Get settings from storage
chrome.storage.sync.get(['enabled'], (result) => {
  if (result.enabled) {
    highlightKeywords();
  }
});

// Listen for messages from popup or background
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'highlight') {
    highlightKeywords();
    sendResponse({ success: true });
  }
});

// Send message to background script
chrome.runtime.sendMessage({
  action: 'pageVisited',
  url: window.location.href
}, (response) => {
  console.log('Background response:', response);
});

// Inject custom UI into page
function injectCustomUI() {
  const container = document.createElement('div');
  container.id = 'my-extension-container';
  container.innerHTML = `
    <div style="position: fixed; top: 10px; right: 10px; z-index: 10000;
                background: white; padding: 10px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1);">
      <h3>My Extension</h3>
      <button id="my-action-btn">Click Me</button>
    </div>
  `;

  document.body.appendChild(container);

  // Add event listener
  document.getElementById('my-action-btn').addEventListener('click', () => {
    alert('Button clicked!');
  });
}

injectCustomUI();
```

### Content Script Isolation

Content scripts have isolated JavaScript environment but share DOM with the page.

```javascript
// ✅ Content script can access/modify DOM
document.querySelector('h1').textContent = 'Modified by extension';

// ✅ Content script has its own JavaScript scope
let myVar = 'extension variable'; // Won't conflict with page variables

// ❌ Cannot access page's JavaScript variables directly
// console.log(window.pageVariable); // Won't work!

// ✅ To communicate with page JavaScript, use window.postMessage
window.postMessage({ type: 'FROM_EXTENSION', data: 'hello' }, '*');

// Listen for responses from page
window.addEventListener('message', (event) => {
  if (event.source === window && event.data.type === 'FROM_PAGE') {
    console.log('Received from page:', event.data);
  }
});
```

---

## Popup Interface

### Popup HTML

**popup/popup.html:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Extension Popup</title>
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div class="container">
    <h1>My Extension</h1>

    <div class="stats">
      <div class="stat-item">
        <span class="stat-label">Pages Visited:</span>
        <span id="pageCount" class="stat-value">0</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">Status:</span>
        <span id="status" class="stat-value">Active</span>
      </div>
    </div>

    <div class="toggle-section">
      <label class="toggle">
        <input type="checkbox" id="enableToggle" checked>
        <span class="slider"></span>
      </label>
      <span>Enable Extension</span>
    </div>

    <button id="actionBtn" class="primary-btn">Perform Action</button>
    <button id="clearBtn" class="secondary-btn">Clear Data</button>

    <div id="message" class="message hidden"></div>
  </div>

  <script src="popup.js"></script>
</body>
</html>
```

### Popup JavaScript

**popup/popup.js:**

```javascript
// Get DOM elements
const pageCountEl = document.getElementById('pageCount');
const statusEl = document.getElementById('status');
const enableToggle = document.getElementById('enableToggle');
const actionBtn = document.getElementById('actionBtn');
const clearBtn = document.getElementById('clearBtn');
const messageEl = document.getElementById('message');

// Load saved data when popup opens
chrome.storage.local.get(['pageCount', 'enabled'], (result) => {
  pageCountEl.textContent = result.pageCount || 0;
  enableToggle.checked = result.enabled !== false; // Default true
  updateStatus(enableToggle.checked);
});

// Toggle extension on/off
enableToggle.addEventListener('change', (e) => {
  const enabled = e.target.checked;

  chrome.storage.sync.set({ enabled }, () => {
    updateStatus(enabled);
    showMessage(enabled ? 'Extension enabled' : 'Extension disabled', 'success');

    // Notify content scripts
    chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
      chrome.tabs.sendMessage(tabs[0].id, {
        action: 'toggleExtension',
        enabled
      });
    });
  });
});

// Action button click
actionBtn.addEventListener('click', async () => {
  actionBtn.disabled = true;
  actionBtn.textContent = 'Processing...';

  try {
    // Send message to background script
    const response = await chrome.runtime.sendMessage({
      action: 'performAction'
    });

    if (response.success) {
      showMessage('Action completed successfully!', 'success');
    } else {
      showMessage('Action failed: ' + response.error, 'error');
    }
  } catch (error) {
    showMessage('Error: ' + error.message, 'error');
  } finally {
    actionBtn.disabled = false;
    actionBtn.textContent = 'Perform Action';
  }
});

// Clear data button
clearBtn.addEventListener('click', () => {
  if (confirm('Are you sure you want to clear all data?')) {
    chrome.storage.local.clear(() => {
      pageCountEl.textContent = '0';
      showMessage('Data cleared successfully', 'success');
    });
  }
});

// Get current tab info
chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
  const currentTab = tabs[0];
  console.log('Current tab:', currentTab.url);

  // You can show tab-specific info in popup
  if (currentTab.url.includes('github.com')) {
    statusEl.textContent = 'GitHub detected';
    statusEl.style.color = 'green';
  }
});

// Helper functions
function updateStatus(enabled) {
  statusEl.textContent = enabled ? 'Active' : 'Inactive';
  statusEl.style.color = enabled ? 'green' : 'red';
}

function showMessage(text, type = 'info') {
  messageEl.textContent = text;
  messageEl.className = `message ${type}`;
  messageEl.classList.remove('hidden');

  setTimeout(() => {
    messageEl.classList.add('hidden');
  }, 3000);
}

// Listen for messages from background
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'updateCount') {
    pageCountEl.textContent = message.count;
  }
});
```

---

## Chrome Storage API

Chrome provides three storage areas: `local`, `sync`, and `session`.

### Storage Patterns

```javascript
// SYNC STORAGE (syncs across devices, 100KB limit)
chrome.storage.sync.set({ key: 'value' }, () => {
  console.log('Saved to sync storage');
});

chrome.storage.sync.get(['key'], (result) => {
  console.log('Value:', result.key);
});

// LOCAL STORAGE (local only, 10MB limit per extension)
chrome.storage.local.set({ largeData: {...} }, () => {
  console.log('Saved to local storage');
});

chrome.storage.local.get(['largeData'], (result) => {
  console.log('Large data:', result.largeData);
});

// SESSION STORAGE (clears when browser closes, 10MB limit)
chrome.storage.session.set({ tempData: 'value' }, () => {
  console.log('Saved to session storage');
});

// Get multiple keys
chrome.storage.sync.get(['key1', 'key2', 'key3'], (result) => {
  console.log(result.key1, result.key2, result.key3);
});

// Get all data
chrome.storage.sync.get(null, (allData) => {
  console.log('All data:', allData);
});

// Remove keys
chrome.storage.sync.remove(['key1', 'key2'], () => {
  console.log('Keys removed');
});

// Clear all
chrome.storage.local.clear(() => {
  console.log('All local storage cleared');
});

// Listen for storage changes
chrome.storage.onChanged.addListener((changes, areaName) => {
  console.log('Storage area changed:', areaName);

  for (let key in changes) {
    const change = changes[key];
    console.log(`${key} changed from ${change.oldValue} to ${change.newValue}`);
  }
});
```

### Storage Helper Utility

**utils/storage.js:**

```javascript
const Storage = {
  // Get with default value
  async get(key, defaultValue = null) {
    return new Promise((resolve) => {
      chrome.storage.local.get([key], (result) => {
        resolve(result[key] !== undefined ? result[key] : defaultValue);
      });
    });
  },

  // Set value
  async set(key, value) {
    return new Promise((resolve) => {
      chrome.storage.local.set({ [key]: value }, resolve);
    });
  },

  // Remove key
  async remove(key) {
    return new Promise((resolve) => {
      chrome.storage.local.remove(key, resolve);
    });
  },

  // Clear all
  async clear() {
    return new Promise((resolve) => {
      chrome.storage.local.clear(resolve);
    });
  },

  // Get multiple keys
  async getMultiple(keys) {
    return new Promise((resolve) => {
      chrome.storage.local.get(keys, resolve);
    });
  },

  // Increment a numeric value
  async increment(key, amount = 1) {
    const current = await this.get(key, 0);
    const newValue = current + amount;
    await this.set(key, newValue);
    return newValue;
  }
};

// Usage
// await Storage.set('count', 10);
// const count = await Storage.get('count', 0);
// await Storage.increment('count');
```

---

## Message Passing

### One-Time Messages

```javascript
// From content script to background
chrome.runtime.sendMessage({
  action: 'getData',
  payload: { id: 123 }
}, (response) => {
  console.log('Response:', response);
});

// From background to content script (specific tab)
chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
  chrome.tabs.sendMessage(tabs[0].id, {
    action: 'updateUI',
    data: { count: 42 }
  }, (response) => {
    console.log('Content script response:', response);
  });
});

// From popup to background
chrome.runtime.sendMessage({
  action: 'getStats'
}, (response) => {
  console.log('Stats:', response);
});
```

### Long-Lived Connections (Port)

For ongoing communication:

```javascript
// Content script: Open port
const port = chrome.runtime.connect({ name: 'myPort' });

port.postMessage({ action: 'subscribe' });

port.onMessage.addListener((message) => {
  console.log('Received:', message);
});

port.onDisconnect.addListener(() => {
  console.log('Port disconnected');
});

// Background: Listen for connections
chrome.runtime.onConnect.addListener((port) => {
  console.log('Port connected:', port.name);

  port.onMessage.addListener((message) => {
    console.log('Received from port:', message);

    // Send back
    port.postMessage({ response: 'acknowledged' });
  });
});
```

---

## Working with Tabs

### Tab Operations

```javascript
// Get current tab
chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
  const currentTab = tabs[0];
  console.log('Current tab URL:', currentTab.url);
});

// Create new tab
chrome.tabs.create({
  url: 'https://example.com',
  active: true // Set to false to open in background
}, (tab) => {
  console.log('Created tab:', tab.id);
});

// Update tab
chrome.tabs.update(tabId, {
  url: 'https://newurl.com',
  active: true
});

// Close tab
chrome.tabs.remove(tabId);

// Reload tab
chrome.tabs.reload(tabId);

// Get all tabs
chrome.tabs.query({}, (tabs) => {
  console.log('All tabs:', tabs.length);
});

// Get tabs by URL pattern
chrome.tabs.query({ url: 'https://github.com/*' }, (tabs) => {
  console.log('GitHub tabs:', tabs);
});

// Duplicate tab
chrome.tabs.duplicate(tabId);

// Move tab to different window
chrome.tabs.move(tabId, { windowId: windowId, index: 0 });

// Group tabs
chrome.tabs.group({ tabIds: [tabId1, tabId2] }, (groupId) => {
  chrome.tabGroups.update(groupId, { title: 'My Group', color: 'blue' });
});
```

### Tab Events

```javascript
// Tab created
chrome.tabs.onCreated.addListener((tab) => {
  console.log('Tab created:', tab.url);
});

// Tab updated (URL change, loading complete, etc.)
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete') {
    console.log('Tab loaded:', tab.url);
  }
});

// Tab activated (switched to)
chrome.tabs.onActivated.addListener((activeInfo) => {
  console.log('Switched to tab:', activeInfo.tabId);
});

// Tab removed
chrome.tabs.onRemoved.addListener((tabId, removeInfo) => {
  console.log('Tab closed:', tabId);
});
```

---

## Context Menus

Add items to right-click menu:

```javascript
// In background service worker
chrome.runtime.onInstalled.addListener(() => {
  // Create context menu item
  chrome.contextMenus.create({
    id: 'myMenuItem',
    title: 'Send to My Extension',
    contexts: ['selection', 'link', 'image', 'page']
  });

  // Create submenu
  chrome.contextMenus.create({
    id: 'parent',
    title: 'My Extension',
    contexts: ['all']
  });

  chrome.contextMenus.create({
    id: 'child1',
    parentId: 'parent',
    title: 'Option 1',
    contexts: ['all']
  });

  chrome.contextMenus.create({
    id: 'child2',
    parentId: 'parent',
    title: 'Option 2',
    contexts: ['all']
  });
});

// Handle clicks
chrome.contextMenus.onClicked.addListener((info, tab) => {
  console.log('Menu item clicked:', info.menuItemId);
  console.log('Selection:', info.selectionText);
  console.log('Link URL:', info.linkUrl);
  console.log('Page URL:', info.pageUrl);

  if (info.menuItemId === 'myMenuItem') {
    // Do something with selected text
    if (info.selectionText) {
      processText(info.selectionText);
    }
  }
});
```

---

## Notifications

```javascript
// Show notification
chrome.notifications.create('myNotificationId', {
  type: 'basic',
  iconUrl: 'assets/icons/icon128.png',
  title: 'Extension Notification',
  message: 'This is a notification from the extension',
  priority: 2
}, (notificationId) => {
  console.log('Notification created:', notificationId);
});

// With buttons
chrome.notifications.create({
  type: 'basic',
  iconUrl: 'assets/icons/icon128.png',
  title: 'Notification with Actions',
  message: 'Choose an action',
  buttons: [
    { title: 'Action 1' },
    { title: 'Action 2' }
  ]
});

// Listen for button clicks
chrome.notifications.onButtonClicked.addListener((notificationId, buttonIndex) => {
  console.log('Button clicked:', buttonIndex);

  if (buttonIndex === 0) {
    // Action 1
  } else {
    // Action 2
  }
});

// Listen for notification clicks
chrome.notifications.onClicked.addListener((notificationId) => {
  console.log('Notification clicked:', notificationId);

  // Open a tab or perform action
  chrome.tabs.create({ url: 'https://example.com' });
});

// Clear notification
chrome.notifications.clear('myNotificationId');
```

---

## Building with Modern Tools

### Using Vite + React/TypeScript

**vite.config.ts:**

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { crx } from '@crxjs/vite-plugin';
import manifest from './manifest.json';

export default defineConfig({
  plugins: [
    react(),
    crx({ manifest })
  ],
  build: {
    rollupOptions: {
      input: {
        popup: 'popup/popup.html',
        options: 'options/options.html'
      }
    }
  }
});
```

**package.json:**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@crxjs/vite-plugin": "^2.0.0",
    "@types/chrome": "^0.0.258",
    "@vitejs/plugin-react": "^4.2.1",
    "typescript": "^5.3.3",
    "vite": "^5.0.8"
  }
}
```

---

## TypeScript Types

Install Chrome types:

```bash
npm install -D @types/chrome
```

**Usage:**

```typescript
// Service worker
chrome.runtime.onInstalled.addListener((details: chrome.runtime.InstalledDetails) => {
  console.log('Installed:', details.reason);
});

// Storage
interface Settings {
  enabled: boolean;
  theme: 'light' | 'dark';
  apiKey?: string;
}

chrome.storage.sync.get(['settings'], (result: { settings?: Settings }) => {
  const settings = result.settings || { enabled: true, theme: 'light' };
  console.log('Settings:', settings);
});

// Tabs
chrome.tabs.query({ active: true }, (tabs: chrome.tabs.Tab[]) => {
  const currentTab = tabs[0];
  console.log('Current URL:', currentTab.url);
});
```

---

## Testing

### Manual Testing

1. **Load unpacked extension:**
   - Go to `chrome://extensions/`
   - Enable "Developer mode"
   - Click "Load unpacked"
   - Select your extension directory

2. **Test all features:**
   - Test popup UI
   - Test content script injection
   - Test background service worker
   - Test message passing
   - Check console for errors

3. **Debug:**
   - Popup: Right-click popup → Inspect
   - Background: Click "service worker" link in extensions page
   - Content script: Inspect page → Console (content script logs appear here)

### Automated Testing

**Using Jest:**

```javascript
// Mock chrome API
global.chrome = {
  storage: {
    local: {
      get: jest.fn(),
      set: jest.fn(),
    },
  },
  runtime: {
    sendMessage: jest.fn(),
    onMessage: {
      addListener: jest.fn(),
    },
  },
};

// Test
test('should save to storage', () => {
  const data = { key: 'value' };
  chrome.storage.local.set(data);

  expect(chrome.storage.local.set).toHaveBeenCalledWith(data);
});
```

---

## Publishing to Chrome Web Store

### Preparation

1. **Create promotional images:**
   - Icon: 128x128px
   - Small tile: 440x280px
   - Marquee: 1400x560px
   - Screenshots: 1280x800px or 640x400px

2. **Create store listing:**
   - Detailed description
   - Category
   - Privacy policy (required if handling user data)

3. **Zip your extension:**
   ```bash
   zip -r my-extension.zip . -x ".*" -x "__MACOSX" -x "node_modules/*"
   ```

4. **Submit:**
   - Go to [Chrome Web Store Developer Dashboard](https://chrome.google.com/webstore/devconsole)
   - Pay one-time $5 developer fee
   - Upload ZIP file
   - Fill in store listing
   - Submit for review

### Review Process

- Usually takes 1-3 business days
- Extensions are reviewed for policy compliance
- Common rejection reasons:
  - Requesting unnecessary permissions
  - Misleading descriptions
  - Violating content policies
  - Security vulnerabilities

---

## Common Patterns

### Singleton Pattern for Service Worker

```javascript
// Ensure only one instance of data processor
class DataProcessor {
  static instance = null;

  constructor() {
    if (DataProcessor.instance) {
      return DataProcessor.instance;
    }

    this.data = [];
    DataProcessor.instance = this;
  }

  addData(item) {
    this.data.push(item);
  }
}

const processor = new DataProcessor();
```

### Debounced Storage Updates

```javascript
let saveTimeout;

function debounceSave(key, value, delay = 1000) {
  clearTimeout(saveTimeout);

  saveTimeout = setTimeout(() => {
    chrome.storage.local.set({ [key]: value });
  }, delay);
}

// Usage: Saves only after user stops typing for 1 second
input.addEventListener('input', (e) => {
  debounceSave('userInput', e.target.value);
});
```

---

## Resources

- **Chrome Extensions Docs:** https://developer.chrome.com/docs/extensions/
- **Manifest V3 Migration:** https://developer.chrome.com/docs/extensions/migrating/
- **Chrome API Reference:** https://developer.chrome.com/docs/extensions/reference/
- **Sample Extensions:** https://github.com/GoogleChrome/chrome-extensions-samples
- **Web Store Publishing:** https://developer.chrome.com/docs/webstore/publish/

---

*This guide covers modern Chrome extension development with Manifest V3. For Firefox or Edge-specific adaptations, consult respective browser documentation.*
