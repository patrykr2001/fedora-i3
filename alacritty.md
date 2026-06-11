# Alacritty configs

## Catppucin mocha theme

Edit config:
```bash
nano ~/.config/alacritty/alacritty.toml
```

Write:
```toml
[window]
padding = { x = 12, y = 12 }
decorations = "none"
opacity = 0.95
blur = true

[font]
size = 12.0

[font.normal]
family = "Iosevka Nerd Font Mono"
style = "Regular"

[font.bold]
family = "Iosevka Nerd Font Mono"
style = "Bold"

[font.italic]
family = "Iosevka Nerd Font Mono"
style = "Italic"

[font.offset]
x = 0
y = 2

[cursor]
style = { shape = "Block", blinking = "On" }
blink_interval = 500
unfocused_hollow = true

[scrolling]
history = 10000
multiplier = 3

[selection]
save_to_clipboard = true

# Catppuccin Mocha
[colors.primary]
background = "#1e1e2e"
foreground = "#cdd6f4"

[colors.cursor]
text = "#1e1e2e"
cursor = "#f5e0dc"

[colors.normal]
black   = "#45475a"
red     = "#f38ba8"
green   = "#a6e3a1"
yellow  = "#f9e2af"
blue    = "#89b4fa"
magenta = "#f5c2e7"
cyan    = "#94e2d5"
white   = "#bac2de"

[colors.bright]
black   = "#585b70"
red     = "#f38ba8"
green   = "#a6e3a1"
yellow  = "#f9e2af"
blue    = "#89b4fa"
magenta = "#f5c2e7"
cyan    = "#94e2d5"
white   = "#a6adc8"

[colors.selection]
text       = "#1e1e2e"
background = "#f5e0dc"

[keyboard]
bindings = [
  { key = "V", mods = "Control|Shift", action = "Paste" },
  { key = "C", mods = "Control|Shift", action = "Copy" },
  { key = "Plus", mods = "Control", action = "IncreaseFontSize" },
  { key = "Minus", mods = "Control", action = "DecreaseFontSize" },
  { key = "Key0", mods = "Control", action = "ResetFontSize" },
]
```

