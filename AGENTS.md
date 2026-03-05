# AIODIALOG-MENUS SKILL KNOWLEDGE BASE

**Generated:** 2026-03-05
**Type:** AI Coding Agent Skill (OpenCode, Claude, Codex, etc.)

## OVERVIEW
Reference skill for building interactive Telegram bot menus using `aiogram_dialog` library. Covers Dialog > Windows > Widgets architecture with Python async patterns.

## STRUCTURE
```
aiogram-dialog-menus/
└── SKILL.md    # Complete reference (451 lines)
```

## WHERE TO LOOK
| Task | Section | Notes |
|------|----------|-------|
| Basic setup | "Setup" | Registry, setup_dialogs(), register handlers |
| Dialog creation | "Creating a Dialog" | Dialog class, Window, getter functions |
| Buttons | "Button Widget" | OnClick, OnEvent, switch_to, start modes |
| Selection widgets | "Select/Multiselect/Radio/Checkbox" | item_id_getter, on_click, on_state_change |
| Layout | "Rows/Columns/ScrollingGroup" | Group, scrolling, keyboard layouts |
| Data flow | "Getters & Data" | getter return dict, dialog_data, start_data |
| Error handling | "Error Handling" | UnknownIntent, UnknownState handlers |
| Migration | "Migration Guide" | v1→v2 changes, reset_stack→StartMode |

## CONVENTIONS
- **Stack**: Python ≥3.9, aiogram ≥3.14, aiogram-dialog v2.x
- **Naming**: snake_case functions/vars, PascalCase classes
- **Style**: async/await throughout, type hints expected
- **Required**: UnknownIntent + UnknownState handlers registered
- **Getters**: Must return `dict` (never `None`), cache expensive ops in `dialog_data`

## ANTI-PATTERNS (THIS SKILL)
| Pattern | Why | Fix |
|---------|-----|-----|
| Simple one-off replies | Dialog overhead unnecessary | Use plain `message.answer()` |
| Complex FSM logic | aiogram_dialog handles state | Use Dialog, not manual FSM |
| Exception propagation | Crashes bot | Catch all, show user-friendly error |
| Getter returns `None` | TypeError | Return `{}` for empty state |
| Blocking ops in getter | Blocks event loop | Use async or cache in dialog_data |
| `dialog_data["key"]` | AttributeError | Use `dialog_data.key` (attribute access) |
| `reset_stack=True` | Deprecated v1 | Use `mode=StartMode.RESET_STACK` |
| `manager.context` | Deprecated v1 | Use `manager.current_context()` |
| Missing widget `id` | Widget not tracked | Always set `id="unique_name"` |
| Wrong `item_id_getter` | Selection fails | Use `lambda x: x.id` or appropriate key |

## UNIQUE STYLES
- **Monolithic skill file**: All content in single SKILL.md vs modular sections
- **Example-driven**: Heavy use of code snippets over prose explanations
- **Telegram-specific**: Assumes aiogram ecosystem knowledge

## COMMANDS
```bash
# Install dependencies
pip install aiogram>=3.14 aiogram-dialog

# Run bot (typical)
python main.py
```

## NOTES
- External docs: https://aiogram-dialog.readthedocs.io
- GitHub: Tishka17/aiogram_dialog
- Widget IDs must be unique within dialog
- `StartMode.RESET_STACK` clears stack for main menu entry
- `StartMode.NORMAL` keeps stack (for sub-dialogs)
- `dialog_data` persists across windows within same dialog
- `start_data` passes context from parent dialog
