# 📜 Visual Memory · 可视记忆 — SillyTavern Extension

[中文](README.md)

Automatically extract events from tavern chats, building a visual long-term memory.

---

## Installation

1. Open the SillyTavern extension panel (the diamond/crystal icon at the top) → Install Extension
2. Enter the plugin URL:
   ```
   https://github.com/janfeise/SillyTavern-Visual-Memory
   ```

Install location: `{ST directory}/data/default-user/extensions/st-event-chronicle/`

---

## Quick Start

1. After installation, auto-extraction is enabled by default — just keep chatting and new events will be generated automatically.

2. For existing chat history, you can batch-generate events manually: open extension settings (the diamond/crystal icon at the top) → find `Visual Memory` → click the "Generate Events" button.

> **Note**: Generating events for a full chat history consumes significant Tokens.

<img src="https://raw.githubusercontent.com/janfeise/Event-Chronicle/main/img/%E9%85%8D%E7%BD%AE-en.png" alt="Settings Panel" width="60%" />

---

## Configuration

Adjust in the ST extension settings panel:

| Setting | Default | Description |
|---|---|---|
| Auto Extraction | On | Whether to automatically extract events from new messages |
| Extraction Threshold | 10 messages | Trigger extraction after N new messages |
| Merge Threshold | 5 events | Trigger merge after N new events |
| Max Tokens | 2048 | Maximum output length for event extraction |
| Batch Slice | 12 msgs/round | Number of messages per round in batch generation |
| Model Settings | Auto (use ST) | Optional override: API source, model name, custom URL, temperature |

> If the API call fails (400), check that ST's API settings are configured. Or manually specify `chat_completion_source` and `model` in the plugin settings.

---

## Data Storage

- **Global Config**: `{ST directory}/data/default-user/settings.json` → `extension_settings['event-chronicle']`
- **Event Data**: `{ST directory}/data/default-user/chats/{chat}.jsonl` → `metadata['event-chronicle']`

> Legacy data stored in `settings.json` will be automatically migrated on upgrade. Events are isolated by `chatId` when switching chats.

---

## Features

- **Auto Event Extraction** — Automatically invoke LLM to extract events after every N new messages
- **Long-term Memory Injection** — Inject chronicle into AI context before generation, preventing early plot amnesia
- **Auto Merge & Dedup** — Automatically organize, deduplicate and reorder after accumulating M new events
- **Timeline Browser** — Open standalone window from the Wand menu, with search, location & importance filters
- **Source Message Traceability** — Click an event to expand source chat bubbles and review the original context
- **Event Editing** — Edit title/summary/importance/participants/location/tags with instant persistence
- **One-click Batch Generation** — Incrementally process chat history in slices to avoid context overflow
- **i18n Support** — 简体中文 / English, auto-switches with ST language setting

---

## Timeline Browser

**Timeline Browser** — Wand menu (magic wand icon, bottom-left) → 📋 Browse Timeline:

![img](https://raw.githubusercontent.com/janfeise/Event-Chronicle/main/img/%E6%B5%8F%E8%A7%88%E6%97%B6%E9%97%B4%E7%BA%BF-en.png)

View the complete timeline for you and your current character in a new window, with search, filtering, and source message expansion:

![img](https://raw.githubusercontent.com/janfeise/Event-Chronicle/main/img/timeline%E9%A1%B5%E9%9D%A2-en.png)

---

## Visual Memory vs RAG

Memory is history events, not retrieved information.

Traditional RAG systems retrieve relevant information from stored text.

Event Chronicle treats memory as history rather than information. Instead of storing conversation chunks and retrieving what is most relevant, it transforms conversations into a timeline of historical events, allowing AI to understand the present through the context of the past.

Memory is history events, not relevant information.

---

## Notes

- Event extraction and merging consume API Tokens
- A token usage warning is shown before batch generation
- Refresh the page after changing model settings
- The clear events action is irreversible — use with caution

---

## More

- [Main Project](https://github.com/janfeise/Event-Chronicle)
- [Dev Documentation](https://github.com/janfeise/Event-Chronicle/blob/main/docs/st-extension-dev.md)
- [Main Project README](../README.md)
