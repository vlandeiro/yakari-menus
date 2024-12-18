Menus for [yakari](https://github.com/vlandeiro/yakari)


# Building Command Menus for Yakari

## 1. Basic Concepts

### What is a Command Menu?

A command menu is a TOML configuration file that defines a hierarchical interface for executing commands. It allows you to:
- Organize related commands into menus
- Define reusable arguments
- Create interactive prompts for command values

> [!NOTE]
> All interactions in the TUI are done through keyboard shortcuts. When defining arguments, commands, or menus, the TOML key is the keyboard shortcut that will trigger that item.

### Basic Structure

A menu file must have a name and can contain arguments, commands, and submenus:

```toml
name = "Example Menu"

# Optional global configuration
[configuration]
sort_arguments = true
sort_commands = true
sort_menus = true

# Arguments available to all commands in this menu
[arguments]
"-v" = { flag = "--verbose" }

# Commands that can be executed
[commands.r]
name = "run"
description = "Run the command"
template = ["tool", "run", { include = "*" }]

# Submenus for organizing related commands
[menus.sub]
name = "Submenu"
```

> [!TIP]
> While configuration is optional, it's good practice to define it explicitly to ensure consistent behavior.

### Menu Components

1. **Arguments** modify how commands behave
   ```toml
   [arguments]
   "-v" = { flag = "--verbose" }     # Press '-v' to toggle
   "-n" = { name = "--name" }        # Press '-n' to set value
   ```

2. **Commands** define actions to execute
   ```toml
   [commands.x]                      # Press 'x' to execute
   name = "execute"
   template = ["tool", { include = "*" }]
   ```

3. **Submenus** group related functionality
   ```toml
   [menus.s]                        # Press 's' to enter submenu
   name = "Settings"
   ```

> [!IMPORTANT]
> Each component's key (like `-v`, `x`, or `s` in the examples above) is the keyboard shortcut that will trigger that item in the TUI.

## 2. Menu Configuration

Each menu can have its own configuration that controls sorting behavior and how arguments are formatted in commands. When a menu doesn't define its own configuration, it inherits from its parent menu.

### Sorting Options

```toml
[configuration]
sort_arguments = true  # Sort arguments alphabetically by shortcut
sort_commands = true   # Sort commands alphabetically by shortcut
sort_menus = true     # Sort submenus alphabetically by shortcut
```

> [!NOTE]
> Sorting defaults to `true`. Set to `false` if you want to preserve the order defined in your TOML file.

### Named Arguments Style

The `named_arguments_style` configuration controls how arguments with values are formatted in the final command:

```toml
[configuration.named_arguments_style]
separator = "equal"    # How to join argument name and value
multi_style = "repeat" # How to handle multiple values
```

#### Separator Style
- `"equal"`: formats as `--name=value`
- `"space"`: formats as `--name value`

#### Multi-Value Style
- `"repeat"`: repeats the argument name for each value
  ```bash
  --file=a --file=b  # with separator="equal"
  --file a --file b  # with separator="space"
  ```
- Any other string: joins values with that string
  ```toml
  multi_style = ","
  # Results in: --file=a,b  or  --file a,b
  ```

> [!TIP]
> Choose the style that matches your target command-line tool. Different tools have different conventions - some use spaces (`cmd --opt value`), others use equals (`cmd --opt=value`), and some support both.

### Configuration Inheritance

Configuration cascades from parent menus to child menus:

```toml
name = "Parent Menu"

# Parent configuration
[configuration]
sort_arguments = true
[configuration.named_arguments_style]
separator = "equal"

# Child menu with no configuration
[menus.sub]
name = "Sub Menu"
# Inherits parent's configuration

# Child menu with custom configuration
[menus.custom]
name = "Custom Menu"
[menus.custom.configuration]
sort_arguments = false  # Overrides parent
```

> [!IMPORTANT]
> When a submenu defines its own configuration, it completely overrides the parent's configuration for that menu and its children.
