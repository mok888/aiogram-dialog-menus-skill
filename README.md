# aiogram-dialog-menus

AI coding assistant skill for building interactive Telegram bot menus using `aiogram_dialog` v2.x. Compatible with OpenCode, Claude, Codex, and other AI coding agents.

## What This Does

Provides expert knowledge for creating:
- Multi-step wizards (registration, booking, configuration)
- Menu systems with nested options
- Polls and surveys with interactive selection
- Admin panels with toggleable settings
- Any complex stateful user interaction

## Installation

### OpenCode

Ask your AI agent:

```
Install the aiogram-dialog-menus skill from https://github.com/mok888/aiogram-dialog-menus-skill into ~/.config/opencode/skills/
```

### Manual

Clone or download this repo to your agent's skills directory:

```bash
# OpenCode
git clone https://github.com/mok888/aiogram-dialog-menus-skill ~/.config/opencode/skills/aiogram-dialog-menus

# Or copy the SKILL.md content directly for agents that support custom instructions
```

## Usage

Once installed, the skill activates automatically when:
- Building Telegram bot menus
- Implementing dialog flows
- Creating interactive bot interfaces
- Working with aiogram_dialog widgets

### Example Prompts

```
"Create a registration wizard for my Telegram bot with 3 steps"
"Build a settings menu with toggles using aiogram_dialog"
"Add a language selector to my bot menu"
"Implement an admin panel with multiselect options"
```

## Prerequisites

- Python 3.9+
- aiogram ≥3.14
- aiogram-dialog v2.x

```bash
pip install aiogram>=3.14 aiogram-dialog
```

## Coverage

| Topic | Description |
|-------|-------------|
| Setup | Registry, setup_dialogs(), handler registration |
| Dialogs | Dialog class, Window, StatesGroup |
| Widgets | Button, Select, Multiselect, Radio, Checkbox |
| Layout | Row, Column, Group, ScrollingGroup |
| Data | Getters, dialog_data, start_data |
| Navigation | switch_to, start, back, done |
| Errors | UnknownIntent, UnknownState handlers |
| Migration | v1 → v2 changes |

## Resources

- [Official Docs](https://aiogram-dialog.readthedocs.io)
- [GitHub](https://github.com/Tishka17/aiogram_dialog)
- [Examples](https://github.com/Tishka17/aiogram_dialog/tree/master/example)
