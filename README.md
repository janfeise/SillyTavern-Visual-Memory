# 📜 Visual Memory · 可视记忆 — SillyTavern 扩展

[English](README_EN.md)

从酒馆对话中自动提取事件，构建可视长期记忆

---

## 安装

1. 打开酒馆扩展面板（顶部三个菱形/晶体图标）→ 安装扩展程序
2. 输入插件 URL：
   ```
   https://github.com/janfeise/SillyTavern-Visual-Memory
   ```

插件安装位置：`{ST 目录}/data/default-user/extensions/st-event-chronicle/`

---

## 快速开始

1. 安装后，插件默认开启自动提取——继续聊天即可，新事件会自动生成

2. 对于已有聊天记录，可手动批量生成：打开扩展设置（顶部三个菱形/晶体图标） → 找到 `Visual Memory` → 点击「生成事件」按钮

> **注意**：为完整聊天历史生成事件会消耗大量 Token ❗

<img src="https://raw.githubusercontent.com/janfeise/Event-Chronicle/main/img/%E9%85%8D%E7%BD%AE-zh.png" alt="配置面板" width="60%" />

---

## 配置

在 ST 扩展设置面板中调整：

| 配置项 | 默认值 | 说明 |
|---|---|---|
| 自动提取 | 开启 | 是否自动从新消息中提取事件 |
| 提取阈值 | 10 条消息 | 累计多少条新消息后触发提取 |
| 合并阈值 | 5 个事件 | 累计多少个新事件后触发合并去重 |
| 最大 Token | 2048 | 事件提取 LLM 输出长度上限 |
| 批量切片 | 12 条/轮 | 批量生成时每轮发送的消息数 |
| 模型设置 | 自动复用 ST | 可选覆盖：API 来源、模型名、自定义 URL、Temperature |

> 如果 API 调用失败 (400)，请检查 ST 的 API 设置是否已配置或在插件设置中手动指定 `chat_completion_source` 和 `model`

---

## 数据存放位置

- **全局配置**：`{ST 目录}/data/default-user/settings.json` → `extension_settings['event-chronicle']`
- **事件数据**：`{ST 目录}/data/default-user/chats/{chat}.jsonl` → `metadata['event-chronicle']`

> 旧版本数据存储在 `settings.json` 中，升级后会自动迁移切换聊天时事件按 `chatId` 自动隔离

---

## 功能

- **自动事件提取** — 累计 N 条新消息自动调用 LLM 提取事件
- **长期记忆注入** — AI 生成回复前注入编年史，避免遗忘早期情节
- **自动合并去重** — 累计 M 条新事件自动整理，去重保序
- **时间线浏览** — Wand 菜单打开独立窗口，支持搜索、地点筛选、重要度筛选
- **来源消息追溯** — 点击事件可展开来源对话气泡，回顾事件发生的上下文
- **事件编辑** — 支持修改标题/摘要/重要性/参与者/地点/标签，即时持久化
- **一键批量生成** — 增量处理历史消息，切片发送避免超上下文
- **国际化支持** — 简体中文 / English，随 ST 语言设置自动切换

---

## 时间线浏览

**时间线浏览器** — Wand 菜单（左下角魔法棒）→ 📋 时间线浏览：

![img](https://raw.githubusercontent.com/janfeise/Event-Chronicle/main/img/%E6%97%B6%E9%97%B4%E7%BA%BF%E6%B5%8F%E8%A7%88-zh.png)

在新窗口查看你与当前角色的完整时间线，支持搜索、筛选和来源消息展开：

![img](https://raw.githubusercontent.com/janfeise/Event-Chronicle/main/img/timeline%E9%A1%B5%E9%9D%A2-zh.png)

---

## Visual Memory vs RAG

`memory` 是历史事件，而非检索信息

传统 RAG 从存储的文本中检索相关内容，而 `Event Chronicle` 将记忆视为**历史**而非**信息**——它不存储对话片段并按相关性检索，而是将对话转化为历史事件编年史，让 AI 通过过去的脉络理解现在

记忆是历史事件，不是相关信息

---

## 注意事项

- 事件提取和合并会消耗 API Token
- 批量生成前会弹出 Token 消耗提醒
- 模型设置变更后建议刷新页面生效
- 清空事件操作不可撤销，请谨慎使用

---

## 更多

- [主项目](https://github.com/janfeise/Event-Chronicle)
- [开发文档](https://github.com/janfeise/Event-Chronicle/blob/main/docs/st-extension-dev.md)
- [主项目 README](../README.md)
