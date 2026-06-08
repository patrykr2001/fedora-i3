# Utwórz folder na fonty
```fish
mkdir -p ~/.local/share/fonts
```

# Pobierz Iosevka Nerd Font
```fish
wget https://github.com/ryanoasis/nerd-fonts/releases/latest/download/Iosevka.zip -P /tmp/
```

# Rozpakuj
```fish
unzip /tmp/Iosevka.zip -d ~/.local/share/fonts/Iosevka/
```

# Odśwież cache fontów
```fish
fc-cache -fv
```

# Sprawdź czy font jest dostępny
```fish
fc-list | grep -i iosevka
```

# Ustaw font w Alacritty `~/.config/alacritty/alacritty.toml`:
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

# Ustaw font w i3 `~/.config/i3/config`:
```fish
font pango:IosevkaTerm Nerd Font Mono 10
```

# LazyVim `~/.config/nvim/lua/plugins/ui.lua` - utwórz plik:

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

# I ustaw font w `~/.config/nvim/lua/config/options.lua`:

```lua
vim.opt.guifont = "Iosevka Nerd Font:h12"
```
