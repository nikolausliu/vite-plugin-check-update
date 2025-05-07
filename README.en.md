# vite-plugin-check-update

English | [简体中文](./README.md)

A Vite plugin for detecting and prompting page updates. When a new version is detected, it automatically prompts users to refresh the page to get the latest content.

## Features

- 🔄 Automatic polling for new versions
- 🔔 Timely page refresh prompts
- ⚙️ Flexible configuration options
- 📦 Static resource change detection
- 🎯 Support for both automatic and manual modes

## Installation

```bash
npm install vite-plugin-check-update -D
```

## Usage

### Automatic Mode (Recommended)

The plugin automatically generates and injects the version detection script into HTML, no additional configuration needed:

```ts
import { defineConfig } from 'vite'
import { vitePluginCheckUpdate } from 'vite-plugin-check-update'

export default defineConfig({
  plugins: [
    vitePluginCheckUpdate({
      // Optional configuration
      pollInterval: 5000, // Set detection interval to 5 seconds
      checkTip: 'New version ready, click OK to update now', // Custom prompt text
    }),
  ],
})
```

### Manual Mode

If you want to customize the version detection logic, you can disable automatic injection by setting `silent: true` and implement the detection script yourself:

```ts
import { defineConfig } from 'vite'
import { vitePluginCheckUpdate } from 'vite-plugin-check-update'

export default defineConfig({
  plugins: [
    vitePluginCheckUpdate({
      silent: true, // Disable automatic injection
      versionFile: 'meta/version.json', // Custom version file path
    }),
  ],
})
```

Then implement the version detection logic in your code, which can be customized according to your business needs, for example:

```ts
// version-check.ts
async function checkUpdate() {
  let currentVersion = '' // Should match the initial version.json build
  const res = await fetch('/version.json?t=' + Date.now())
  const { version } = await res.json()
  currentVersion = version

  async function check() {
    try {
      const res = await fetch('/version.json?t=' + Date.now())
      const { version } = await res.json()
      if (version !== currentVersion) {
        currentVersion = version
        showUpdateNotification()
      }
    } catch (err) {
      console.error('Update check failed:', err)
    }
  }

  function showUpdateNotification() {
    const confirmed = confirm('New version found, click OK to refresh')
    if (confirmed) location.reload()
  }

  // Check every 5 minutes
  setInterval(check, 60 * 1000)
}
checkUpdate()
```

## Configuration Options

```ts
interface vitePluginCheckUpdateOptions {
  // Version file path, relative to dist directory
  versionFile?: string // Default: 'version.json'
  // Version information key name
  versionKey?: string // Default: 'version'
  // Hash algorithm
  hashAlgorithm?: string // Default: 'md5'
  // Hash encoding
  encoding?: string // Default: 'hex'
  // Whether to disable automatic injection of detection script
  silent?: boolean // Default: false
  // Detection script file name (only valid when silent is false)
  checkScriptFile?: string // Default: 'version-check.js'
  // Polling interval (milliseconds) (only valid when silent is false)
  pollInterval?: number // Default: 10000
  // Update prompt text (only valid when silent is false)
  checkTip?: string // Default: 'New version found, refresh now to get the latest content?'
}
```

## How It Works

### Build Phase

1. Calculate hash values for `index.html` and `public` directory to generate version information
2. Write version information to the specified version file (default: `version.json`)

### Runtime

- **Automatic Mode**: Plugin automatically injects detection script, periodically polls version file, prompts user to refresh when new version is found
- **Manual Mode**: Only generates version file, users implement detection logic and update prompts themselves

## Development

```bash
# Install dependencies
npm install

# Development mode
npm run dev

# Build
npm run build

# Release
npm run release
```

## License

MIT
