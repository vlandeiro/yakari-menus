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

## 3. Defining Arguments

Arguments modify how commands behave. Each argument is defined by a keyboard shortcut and its properties:

### Flag Arguments

Flag arguments are simple on/off switches:

```toml
[arguments]
"-v" = { flag = "--verbose" }
"-f" = { flag = "--force", description = "Force operation" }
"-a" = { flag = "--all", on = true }  # Enabled by default
```

### Named Arguments

Named arguments accept values:

```toml
[arguments."-n"]
name = "--name"
value = "default"
description = "Your name"

# Password input (hidden when typing)
[arguments."-p"]
name = "--password"
password = true

# Multi-value argument
[arguments."-t"]
name = "--tag"
multi = true
```

> [!NOTE]
> Named arguments that don't start with `-` are treated as positional arguments. They appear in the command without their name (just the value).

### Arguments with Choices

When an argument should only accept specific values:

```toml
[arguments."-m"]
name = "--mode"
choices = ["fast", "slow", "auto"]
selected = "fast"

# Multi-choice argument
[arguments."-l"]
name = "--log-level"
choices = ["debug", "info", "warn", "error"]
multi = true
```

### Argument Suggestions

Arguments can provide suggestions to help users choose values:

```toml
# Static suggestions
[arguments."-e"]
name = "--env"
suggestions = { values = ["dev", "staging", "prod"] }

# Dynamic suggestions from command output
[arguments."-b"]
name = "--branch"
suggestions = { command = "git for-each-ref --format='%(refname:short)' refs/heads/", cache = true }
```

> [!IMPORTANT]
> Dynamic suggestions must come from commands that output exactly one suggestion per line.

### Custom Argument Formatting

By default, arguments follow the menu's `named_arguments_style` configuration. For special cases, you can override this with a template:

```toml
[arguments."-m"]
name = "message"
template = ["-m", "{self.value}"]  # Results in: -m "user message"
```

> [!TIP]
> Templates are useful when an argument doesn't follow the standard `--name=value` or `--name value` pattern.

## 4. Defining Commands

Commands define the actions to execute. Each command is identified by a keyboard shortcut:

```toml
[commands.l]  # Press 'l' to execute this command
name = "list"
description = "List all items"
template = ["ls", "-l", { include = "*" }]
```

### Command Templates

A template defines how to build the final command. It can contain:
- Static strings: `"git"`
- Menu arguments: `{ include = "*" }`
- Dynamic values: `{ name = "message" }`

```toml
[commands.c]
name = "commit"
description = "Create a commit"
template = [
    "git",
    "commit",
    { include = "*" },              # Include enabled menu arguments
    { name = "commit message", template = "-m {self.value}" }     # Prompt user for value
]
```

### Including Arguments

Control which menu arguments to include:

```toml
# Include all enabled arguments
template = ["cmd", { include = "*" }]

# Include specific arguments
template = ["cmd", { include = ["-v", "-f"] }]

# Include all except specific arguments
template = ["cmd", { include = "*", exclude = ["-p"] }]

# Only include arguments from current menu (ignore parent menu arguments)
template = ["cmd", { include = "*", scope = "this" }]
```

> [!NOTE]
> The default scope is "all", which includes arguments from parent menus.

## 5. Menu Organization

Menus can be organized hierarchically to group related commands:

```toml
name = "Main Menu"

# Top-level arguments available to all submenus
[arguments]
"-v" = { flag = "--verbose" }

# Submenu for branch operations
[menus.b]
name = "Branch Operations"

[menus.b.arguments]
"-f" = { flag = "--force" }

[menus.b.commands.c]
name = "create"
template = ["git", "branch", { include = "*" }]

# Another submenu for commit operations
[menus.c]
name = "Commit Operations"

[menus.c.commands.m]
name = "commit"
template = ["git", "commit", { include = "*" }, { name = "commit message", template = "-m {self.value}" }]
```

> [!IMPORTANT]
> Arguments defined in a parent menu are available to all commands in child menus when using `{ include = "*", scope = "all" }`.

