---
name: aiogram-dialog-menus
description: Use when building interactive Telegram bot menus using aiogram_dialog library - creating dialogs with windows, buttons, selectors, and state management for Telegram bots
---

# aiogram_dialog Menu Building Skill

Expert knowledge for building interactive Telegram bot menus using the `aiogram_dialog` library (v2.x).

## When to Use This Skill

**Use for:**
- Multi-step wizards (registration, booking, configuration)
- Menu systems with nested options
- Polls and surveys with interactive selection
- Admin panels with toggleable settings
- Any complex stateful user interaction

**Don't use for:**
- Simple one-off replies (use plain `message.answer()`)
- Complex FSM logic that doesn't fit dialog pattern

## Prerequisites

```bash
pip install aiogram>=3.14 aiogram-dialog
```

- Python 3.9+
- aiogram 3.14+
- aiogram-dialog 2.x

## Core Concepts

### Architecture: Dialog > Windows > Widgets

```
Dialog (manages state + navigation)
  └── Windows (individual screens)
        └── Widgets (interactive elements)
```

### Key Components

1. **StatesGroup** - Define dialog states
2. **Dialog** - Container for windows
3. **Window** - Single screen with widgets
4. **Widgets** - Buttons, selectors, inputs
5. **Getters** - Async functions providing data to windows

## Quick Start

### 1. Setup

```python
from aiogram import Dispatcher
from aiogram_dialog import DialogManager, setup_dialogs

dp = Dispatcher()
setup_dialogs(dp)  # Required once at startup
```

### 2. Define States

```python
from aiogram_dialog import Dialog, Window
from aiogram_dialog.widgets.text import Const
from aiogram_dialog.widgets.kbd import Button
from aiogram.fsm.state import StatesGroup, State

class MainMenu(StatesGroup):
    main = State()
    settings = State()
    profile = State()
```

### 3. Create Window with Button

```python
from aiogram_dialog import StartMode

async def on_settings_click(callback, button, manager: DialogManager):
    await manager.switch_to(MainMenu.settings)

main_window = Window(
    Const("Welcome to the bot!"),
    Button(Const("⚙️ Settings"), id="settings_btn", on_click=on_settings_click),
    state=MainMenu.main,
)
```

### 4. Create Dialog

```python
dialog = Dialog(main_window)
dialog.register(dp, None, MainMenu.main)
```

### 5. Start Dialog

```python
from aiogram_dialog import StartMode

async def start_handler(message, dialog_manager: DialogManager):
    await dialog_manager.start(MainMenu.main, mode=StartMode.RESET_STACK)
```

## Widget Reference

### Button

```python
from aiogram_dialog.widgets.kbd import Button
from aiogram_dialog.widgets.text import Const

# Basic button
Button(Const("Click me"), id="btn1", on_click=my_handler)

# Switch to another window
Button(
    Const("Go to settings"),
    id="to_settings",
    on_click=lambda c, b, m: m.switch_to(MainMenu.settings)
)

# Start another dialog
Button(
    Const("Open profile"),
    id="open_profile",
    on_click=lambda c, b, m: m.start(ProfileDialog.profile)
)
```

### Select (Single Choice)

```python
from aiogram_dialog.widgets.kbd import Select

async def get_items(**kwargs):
    return {"items": ["Option A", "Option B", "Option C"]}

async def on_item_selected(callback, select, manager: DialogManager, item_id: str):
    # item_id is the selected value
    await callback.answer(f"Selected: {item_id}")

Select(
    Const("{item}"),  # Text template
    id="item_select",
    item_id_getter=lambda x: x,  # Extract ID from item
    items="items",  # Key in getter data
    on_click=on_item_selected,
)
```

### Multiselect

```python
from aiogram_dialog.widgets.kbd import Multiselect

async def get_options(**kwargs):
    return {
        "options": [
            {"id": "opt1", "name": "Option 1"},
            {"id": "opt2", "name": "Option 2"},
        ]
    }

Multiselect(
    Const("✓ {item[name]}"),  # Checked format
    Const("  {item[name]}"),  # Unchecked format
    id="multi_select",
    item_id_getter=lambda x: x["id"],
    items="options",
)
```

### Radio

```python
from aiogram_dialog.widgets.kbd import Radio

Radio(
    Const("◉ {item}"),  # Selected
    Const("○ {item}"),  # Unselected
    id="radio_group",
    item_id_getter=lambda x: x,
    items="choices",
)
```

### Checkbox

```python
from aiogram_dialog.widgets.kbd import Checkbox

Checkbox(
    Const("✓ Enabled"),
    Const("○ Disabled"),
    id="feature_toggle",
    default=False,
)
```

### ScrollingGroup

```python
from aiogram_dialog.widgets.kbd import ScrollingGroup

ScrollingGroup(
    Select(
        Const("{item}"),
        id="scroll_select",
        item_id_getter=lambda x: x,
        items="long_list",
    ),
    id="scroll_group",
    width=1,
    height=5,  # Show 5 items at a time
)
```

## Layout Widgets

### Rows and Columns

```python
from aiogram_dialog.widgets.kbd import Row, Column, Group

# Horizontal layout
Row(
    Button(Const("A"), id="a"),
    Button(Const("B"), id="b"),
    Button(Const("C"), id="c"),
)

# Vertical layout
Column(
    Button(Const("1"), id="1"),
    Button(Const("2"), id="2"),
)

# Group with width
Group(
    Button(Const("1"), id="1"),
    Button(Const("2"), id="2"),
    Button(Const("3"), id="3"),
    Button(Const("4"), id="4"),
    width=2,  # 2 buttons per row
)
```

## Data Flow

### Getters (Window Data Provider)

```python
async def user_getter(dialog_manager: DialogManager, **kwargs):
    user_id = dialog_manager.event.from_user.id
    user = await get_user(user_id)
    return {
        "username": user.name,
        "balance": user.balance,
        "level": user.level,
    }

profile_window = Window(
    Const("👤 Profile\n\nName: {username}\nBalance: {balance}"),
    getter=user_getter,  # Attach getter
    state=Profile.view,
)
```

### dialog_data (Persistent State)

```python
# Store data across windows in same dialog
await manager.dialog_data.update({"step": 1, "selected": []})

# Retrieve data
step = manager.dialog_data.get("step", 0)
selected = manager.dialog_data.get("selected", [])

# IMPORTANT: Use attribute access, not dict-style
manager.dialog_data.my_value  # ✓ Correct
manager.dialog_data["my_value"]  # ✗ Wrong (AttributeError)
```

### start_data (From Parent Dialog)

```python
async def open_sub_dialog(callback, button, manager: DialogManager):
    await manager.start(
        SubDialog.main,
        data={"parent_id": 123, "mode": "edit"}  # start_data
    )

# In sub-dialog getter
async def sub_getter(dialog_manager: DialogManager, **kwargs):
    parent_id = dialog_manager.start_data.get("parent_id")
    return {"parent_id": parent_id}
```

## Error Handling

### Required: Unknown Handlers

```python
from aiogram_dialog import Dialog, DialogManager
from aiogram_dialog.widgets.text import Const

dialog = Dialog(
    Window(Const("Main menu"), state=MainMenu.main),
    on_process_unknown=Const("Unknown action. Please use the menu."),
)

# Register unknown handlers
dp.unknown_handler.register(unknown_callback_handler)
```

### UnknownIntent & UnknownState

```python
from aiogram_dialog import Dialog, UnknownIntent, UnknownState

dialog = Dialog(
    # ... windows ...
    on_process_unknown_intent=handle_unknown_intent,
    on_process_unknown_state=handle_unknown_state,
)

async def handle_unknown_intent(update, dialog, manager: DialogManager):
    await manager.switch_to(MainMenu.main)
    if update.message:
        await update.message.answer("Please use the menu buttons.")
```

## Common Patterns

### Main Menu Pattern

```python
class MainMenu(StatesGroup):
    main = State()

main_dialog = Dialog(
    Window(
        Const("🏠 Main Menu\n\nChoose an option:"),
        Row(
            Button(Const("👤 Profile"), id="profile"),
            Button(Const("⚙️ Settings"), id="settings"),
        ),
        Row(
            Button(Const("📊 Stats"), id="stats"),
            Button(Const("❓ Help"), id="help"),
        ),
        state=MainMenu.main,
    )
)

# Always start main menu with RESET_STACK
await dialog_manager.start(MainMenu.main, mode=StartMode.RESET_STACK)
```

### Wizard Pattern

```python
class Registration(StatesGroup):
    name = State()
    email = State()
    confirm = State()

async def on_name_entered(message, widget, manager: DialogManager):
    manager.dialog_data.name = message.text
    await manager.next()

async def on_email_entered(message, widget, manager: DialogManager):
    manager.dialog_data.email = message.text
    await manager.next()

registration_dialog = Dialog(
    Window(
        Const("📝 Step 1: Enter your name"),
        TextInput(id="name_input", on_success=on_name_entered),
        state=Registration.name,
    ),
    Window(
        Const("📧 Step 2: Enter your email"),
        TextInput(id="email_input", on_success=on_email_entered),
        state=Registration.email,
    ),
    Window(
        Const("✅ Confirm:\nName: {dialog_data[name]}\nEmail: {dialog_data[email]}"),
        Button(Const("Confirm"), id="confirm"),
        getter=lambda **kw: {"dialog_data": kw["dialog_manager"].dialog_data},
        state=Registration.confirm,
    ),
)
```

### Admin Panel Pattern

```python
class AdminPanel(StatesGroup):
    main = State()

async def get_admin_stats(**kwargs):
    return {
        "users_count": await db.get_users_count(),
        "messages_count": await db.get_messages_count(),
        "settings": [
            {"id": "maintenance", "name": "Maintenance Mode", "enabled": False},
            {"id": "debug", "name": "Debug Mode", "enabled": True},
        ]
    }

admin_dialog = Dialog(
    Window(
        Const("🔐 Admin Panel\n\nUsers: {users_count}\nMessages: {messages_count}"),
        Multiselect(
            Const("✓ {item[name]}"),
            Const("○ {item[name]}"),
            id="admin_toggles",
            item_id_getter=lambda x: x["id"],
            items="settings",
            on_state_changed=save_admin_settings,
        ),
        getter=get_admin_stats,
        state=AdminPanel.main,
    )
)
```

## Best Practices

### Widget IDs

```python
# ALWAYS set unique IDs for widgets
Button(Const("Click"), id="unique_button_id")  # ✓

# Widget IDs must be unique within the dialog
# They're used for state tracking and callbacks
```

### Getter Performance

```python
# ✓ Cache expensive operations
async def getter(dialog_manager: DialogManager, **kwargs):
    if "cached_data" not in manager.dialog_data:
        data = await expensive_api_call()
        manager.dialog_data.cached_data = data
    return {"data": manager.dialog_data.cached_data}

# ✓ Always return dict
async def good_getter(**kwargs):
    return {"items": []}  # ✓

# ✗ Never return None
async def bad_getter(**kwargs):
    return None  # ✗ Causes TypeError
```

### Exception Handling

```python
async def safe_button_click(callback, button, manager: DialogManager):
    try:
        await risky_operation()
    except Exception as e:
        logger.error(f"Button error: {e}")
        await callback.answer("❌ An error occurred", show_alert=True)
        # Don't let exceptions propagate - they crash the bot
```

## Migration from v1 to v2

| v1 | v2 |
|----|----|
| `reset_stack=True` | `mode=StartMode.RESET_STACK` |
| `manager.context` | `manager.current_context()` |
| `manager.current_stack()` | `manager.stack()` |

## Common Mistakes

1. **Forgetting `setup_dialogs(dp)`** - Dialogs won't work
2. **Missing widget `id`** - Widget state not tracked
3. **Getter returning `None`** - TypeError
4. **Using `dialog_data["key"]`** - Use `dialog_data.key` (attribute)
5. **Not using `StartMode.RESET_STACK`** for main menus - Stack overflow
6. **Wrong `item_id_getter`** - Selection fails
7. **Exceptions in callbacks** - Crashes bot
8. **Blocking operations in getter** - Blocks event loop

## Resources

- **Official Docs**: https://aiogram-dialog.readthedocs.io
- **GitHub**: https://github.com/Tishka17/aiogram_dialog
- **Examples**: https://github.com/Tishka17/aiogram_dialog/tree/master/example

## Quick Reference

```python
# Start dialog (clear stack)
await manager.start(MyDialog.main, mode=StartMode.RESET_STACK)

# Start dialog (keep stack)
await manager.start(SubDialog.main, mode=StartMode.NORMAL)

# Switch window
await manager.switch_to(MyDialog.settings)

# Next/previous window
await manager.next()
await manager.back()

# Update current window
await manager.update()

# End dialog
await manager.done()
```
