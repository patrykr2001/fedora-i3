# Create folder for fonts

```bash
mkdir -p ~/.local/share/fonts
```

# Download your nerd font of choice

E.g. Iosevka Nerd Font
```bash
wget https://github.com/ryanoasis/nerd-fonts/releases/latest/download/Iosevka.zip -P /tmp/
```

# Unzip
```bash
unzip /tmp/Iosevka.zip -d ~/.local/share/fonts/Iosevka/
```

# Refresh fonts cache
```bash
fc-cache -fv
```

# Check if font is available
```bash
fc-list | grep -i iosevka
```

# Set font in Alacritty `~/.config/alacritty/alacritty.toml`:
```toml
[font]
size = 12.0

[font.normal]
family = "IosevkaTerm Nerd Font Mono"
style = "Regular"

[font.bold]
family = "IosevkaTerm Nerd Font Mono"
style = "Bold"


[font.italic]
family = "IosevkaTerm Nerd Font Mono"
style = "Italic"
```

# Set font in i3 `~/.config/i3/config`:
```bash
font pango:IosevkaTerm Nerd Font Mono 10
```

# LazyVim `~/.config/nvim/lua/plugins/ui.lua` - create file:

```lua
return {
  {
    "nvim-lualine/lualine.nvim",
    opts = {
      options = {
        icons_enabled = true,
      },
    },
  },
  {
    "nvim-neo-tree/neo-tree.nvim",
    opts = {
      default_component_configs = {
        icon = { folder_closed = "", folder_open = "", folder_empty = "" },
      },
    },
  },
}
```

# And set font in `~/.config/nvim/lua/config/options.lua`:

```lua
vim.opt.guifont = "Iosevka Nerd Font:h12"
```
