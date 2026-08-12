# Rofi Menus

[Rofi](https://github.com/davatorium/rofi) is a window switcher and application launcher for Linux that doubles as a searchable menu over any list of options. This repo uses it to build:

- **App launcher**: starts installed applications
- **Emoji picker**: searches from a list of emojis and types the selection into the focused window
- **Power menu**: offers lockscreen, shut down, reboot, log out, suspend, and hibernate

They are intended to be mapped to global keyboard shortcuts by the system's window manager.

## Installation

Clone or place this repository anywhere, then symlink the three entry points into a directory on your `PATH`:
```bash
ln -sf <dir>/launcher/launcher.sh   ~/.local/bin/rofi-launcher
ln -sf <dir>/emoji/picker.sh        ~/.local/bin/rofi-emoji
ln -sf <dir>/powermenu/powermenu.sh ~/.local/bin/rofi-powermenu
```
Callers invoke the bare names, so nothing outside this repo hardcodes its location.

Then point `LOCKSCREEN_CMD` at a script that locks the display, somewhere the graphical session sources rather than a shell rc file:
```bash
# ~/.profile
export LOCKSCREEN_CMD=my-lock-script
```

Without it the power menu leaves out Lock, Suspend, and Hibernate rather than offering actions that would fail. See `powermenu/README.md`, which also covers overriding the log-out command for a given window manager.

Requires:

- `rofi` for all three, developed against 1.7.1
- `xdotool`, `xclip` for the emoji picker, to type the selection and copy it
- `systemctl`, `loginctl` for the power menu, for the system and session actions

## Shared Design

### File Structure

All three tools share a palette file and a base theme:

```
rofi-menus/
├── colors/
│   └── kanagawa-dragon.rasi   # shared palette (6 variables)
├── shared-style.rasi          # base theme; imported by all three tools
├── launcher/
│   ├── launcher.sh
│   └── style.rasi
├── emoji/
│   ├── picker.sh
│   ├── emojis.txt
│   └── style.rasi
└── powermenu/
    ├── powermenu.sh
    └── style.rasi
```

### Theme Architecture

`shared-style.rasi` is the base theme imported by all three tools. It imports `kanagawa-dragon.rasi` for color variables and defines the font, window, mainbox, inputbar, prompt, entry, listview, element, element-text, and element-selected rules. Each tool has its own `style.rasi` that imports `shared-style.rasi` and overrides only the rules specific to it. Any change to the base visual language propagates to all three tools automatically. The `configuration {}` block for each tool lives in its own `style.rasi` rather than in a global `config.rasi`, keeping each tool's rofi settings co-located with its theme. Nothing here is loaded by rofi implicitly: every invocation passes `-theme` explicitly, which is what lets the repo live anywhere rather than under `~/.config/rofi`.

All imports are relative to the importing file, so the tree can be relocated as a unit.

> [!Note] 
> Any edit to `shared-style.rasi` or `kanagawa-dragon.rasi` should be followed by opening all three tools to confirm none regressed visually.
