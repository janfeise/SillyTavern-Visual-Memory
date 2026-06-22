# Event Chronicle ST Extension — CLAUDE.md

## Build & Sync

```bash
# From parent project:
npm run sync:st        # Build browser SDK + copy to lib/ec-sdk.mjs
npm run sync:st:push   # Sync + commit + git subtree push

# Or push directly from st-extension/:
git push origin main
```

## Architecture

```
index.js (ST 集成层)
  ├── init()           # 生命周期: 缓存 API 配置, 注入 LLM 通道, 注入 UI, 注册钩子
  ├── llmCall()        # LLM 调用: POST /api/backends/chat-completions/generate
  ├── injectSettingsUI # 动态创建设置面板
  └── setupWandMenu()  # Wand 菜单入口
        │
        ▼
ec-bridge.js (适配层)
  ├── extractEvents()  # Phase 1: 格式化 → 构建 prompt → LLM → 解析
  ├── mergeEvents()    # Phase 2: 去重/合并
  ├── processMessages()# 完整流程编排
  └── startBatchGeneration()  # 历史消息批量回填
        │
        ▼
lib/ec-sdk.mjs (SDK 纯函数, 由 npm run sync:st 生成)
  parseEvents, formatMessages, parseInstructions, applyInstructions,
  extractPrompt, mergePrompt, memoryPrompt, ...
```

## Event Hooks

| Hook | 触发 | 行为 |
|---|---|---|
| `CHARACTER_MESSAGE_RENDERED` | 角色回复后 | 累计未处理消息达阈值 → 自动提取事件 |
| `CHAT_CHANGED` | 切换对话 | 重置追踪状态, 重新注入 prompt |
| `MESSAGE_DELETED` / `MESSAGE_UPDATED` | 消息变更 | 重置追踪标记 |

## Storage

```
extension_settings['event-chronicle']
  ├── _events   # { chatId: Event[] }
  ├── _merge    # { chatId: { newEventCount, lastMergeAt } }
  └── _batch    # { chatId: { lastProcessedIndex, completed } }
```

数据随 ST 的 `settings.json` 自动持久化，无独立文件。

## Key Files

| File | Lines | Purpose |
|---|---|---|
| `index.js` | ~700 | ST 生命周期, LLM 桥接, 设置 UI, 公共 API |
| `ec-bridge.js` | ~720 | SDK 适配, 提取/合并流程, 批量生成, CRUD |
| `lib/ec-sdk.mjs` | ~470 | SDK 纯函数 bundle (generated, do not edit) |
| `timeline.html` + `timeline.js` + `editor.js` | — | 独立时间线浏览器窗口 |
| `settings.html` | ~115 | 设置面板模板 (iframe 加载) |
| `manifest.json` | ~14 | ST 扩展声明 + i18n 声明 |
| `i18n/zh-cn.json` | ~111 | 简体中文语言包 |
| `i18n/en.json` | ~111 | English language pack |

## 国际化 (i18n)

采用 ST 官方 `manifest.json` i18n 方案，语言文件在扩展激活前自动加载。

### 目录结构

```
st-extension/
├── i18n/
│   ├── zh-cn.json    # 简体中文（默认）
│   └── en.json       # English
└── manifest.json     # i18n 字段声明
```

### manifest.json

```json
{
  "i18n": {
    "zh-cn": "i18n/zh-cn.json",
    "en": "i18n/en.json"
  }
}
```

ST 启动时自动 fetch 并调用 `addLocaleData()`，扩展代码无需手动加载。

### 翻译 API

| API | 用途 | 获取方式 |
|---|---|---|
| `translate(key)` | 按语义 key 查翻译 | `getContext().translate` |
| `` t`text ${val}` `` | 标签模板字面量（带变量） | `getContext().t` |
| `data-i18n="key"` | HTML 属性自动翻译 | ST MutationObserver |

**重要**：`t` 是标签模板字面量，不是普通函数。`t('key')` 是错误用法。

### 代码中的使用

**index.js**（主入口）：
```javascript
// 顶部封装 helper
let _translate = null;
function initI18n() {
  const ctx = getContext();
  _translate = ctx.translate || ((key) => key);
}
function tr(key) { return _translate(key); }

// init() 中调用
initI18n();
ecBridge.setTranslate(tr);

// 使用
toastr.info(tr("ec.toast.noNewEvents"), "Event Chronicle");
toastr.success(_t`📜 提取完成 — 新增 ${count} 个事件`, "Event Chronicle");
```

**ec-bridge.js**（适配层）：
```javascript
// 通过依赖注入获取（与 _generateRaw 同模式）
let _translate = (key) => key;
export function setTranslate(fn) { _translate = fn; }
function tr(key) { return _translate(key); }
```

**timeline.js / editor.js**（独立页面）：
```javascript
function tr(key) {
  var a = api();  // window.opener.EventChronicle
  if (a && a.translate) return a.translate(key);
  return key;
}
```

**timeline.html**（静态 HTML）：
```html
<button data-i18n="ec.timeline.refresh">刷新</button>
<input data-i18n="[placeholder]ec.timeline.searchPlaceholder" placeholder="搜索事件..." />
```

ST 的 MutationObserver 自动翻译通过 `innerHTML` 动态注入的带 `data-i18n` 的元素。

### Key 命名规范

```
ec.{module}.{feature}[.{detail}]
```

模块：`settings` / `toast` / `dialog` / `timeline` / `menu` / `status` / `notice` / `memory` / `editor` / `export`

带变量的字符串用 `${0}`、`${1}` 占位符（ST 标签模板索引格式）。

### 扩展新语言

1. 创建 `i18n/xx-xx.json`（复制 en.json 修改 value）
2. `manifest.json` 的 `i18n` 中加一行：`"xx-xx": "i18n/xx-xx.json"`

### 不国际化的部分

- `console.log` 开发日志（用户不可见）
- `manifest.json` 的 `display_name`（ST 不翻译）
- SVG 图标
- 品牌名 "Visual Memory"

## Testing

```bash
# Validate SDK bundle loads in Node (no browser needed):
node -e "import('./lib/ec-sdk.mjs').then(m => console.log(Object.keys(m)))"

# Or via parent project:
npm run test:browser   # Start server → open in browser
node test/browser/run-tests.mjs  # Headless Playwright
```
