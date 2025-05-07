# vite-plugin-check-update

[English](./README.en.md) | 简体中文

一个用于检测和提示页面更新的 Vite 插件。当检测到新版本时，会自动提示用户刷新页面以获取最新内容。

## 功能特点

- 🔄 自动轮询检测新版本
- 🔔 及时提示用户刷新页面
- ⚙️ 灵活的配置选项
- 📦 支持检测静态资源变化
- 🎯 支持自动和手动两种使用模式

## 安装

```bash
npm install vite-plugin-check-update -D
```

## 使用方法

### 自动模式（推荐）

插件会自动生成版本检测脚本并注入到 HTML 中，无需额外配置：

```ts
import { defineConfig } from 'vite'
import { vitePluginCheckUpdate } from 'vite-plugin-check-update'

export default defineConfig({
  plugins: [
    vitePluginCheckUpdate({
      // 可选配置
      pollInterval: 5000, // 设置检测间隔为5秒
      checkTip: '新版本已就绪，点击确定立即更新', // 自定义提示文本
    }),
  ],
})
```

### 手动模式

如果你想自定义版本检测逻辑，可以通过设置 `silent: true` 来禁用自动注入，然后自行实现检测脚本：

```ts
import { defineConfig } from 'vite'
import { vitePluginCheckUpdate } from 'vite-plugin-check-update'

export default defineConfig({
  plugins: [
    vitePluginCheckUpdate({
      silent: true, // 禁用自动注入
      versionFile: 'meta/version.json', // 自定义版本文件路径
    }),
  ],
})
```

然后在你的代码中实现版本检测逻辑，这部分可以根据业务来自行实现，例如：

```ts
// version-check.ts
async function checkUpdate() {
  let currentVersion = '' // 需与首次构建的 version.json 一致
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
      console.error('更新检查失败:', err)
    }
  }

  function showUpdateNotification() {
    const confirmed = confirm('发现新版本，点击确定刷新页面')
    if (confirmed) location.reload()
  }

  // 每5分钟检查一次
  setInterval(check, 60 * 1000)
}
checkUpdate()
```

## 配置选项

```ts
interface VitePluginCheckUpdateOptions {
  // 版本文件路径，相对于 dist 目录
  versionFile?: string // 默认: 'version.json'
  // 版本信息的键名
  versionKey?: string // 默认: 'version'
  // 哈希算法
  hashAlgorithm?: string // 默认: 'md5'
  // 哈希编码
  encoding?: string // 默认: 'hex'
  // 是否禁用自动注入检测脚本
  silent?: boolean // 默认: false
  // 检测脚本文件名（仅在 silent 为 false 时有效）
  checkScriptFile?: string // 默认: 'version-check.js'
  // 轮询间隔（毫秒）（仅在 silent 为 false 时有效）
  pollInterval?: number // 默认: 10000
  // 更新提示文本（仅在 silent 为 false 时有效）
  checkTip?: string // 默认: '发现新版本，是否立即刷新获取最新内容？'
}
```

## 工作原理

### 构建阶段

1. 计算 `index.html` 和 `public` 目录的哈希值，生成版本信息
2. 将版本信息写入指定的版本文件（默认为 `version.json`）

### 运行时

- **自动模式**：插件自动注入检测脚本，定期轮询检查版本文件，发现新版本时提示用户刷新
- **手动模式**：仅生成版本文件，由用户自行实现检测逻辑和更新提示

## 开发

```bash
# 安装依赖
npm install

# 开发模式
npm run dev

# 构建
npm run build

# 发布
npm run release
```

## License

MIT
