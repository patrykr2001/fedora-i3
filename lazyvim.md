# Install

# Install neovim

```bash
sudo dnf install neovim
```

# Install lazyvim dependencies

```bash
sudo dnf install git gcc make ripgrep fd-find
```

# Install LazyVim

```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

# Optionally edit default colorscheme:

```bash
nano ~/.config/nvim/lua/plugins/colorscheme.lua
```

And put:
```bash
return {
  {
    "catppuccin/nvim",
    name = "catppuccin",
    opts = {
      flavour = "mocha",
    },
  },
  {
    "LazyVim/LazyVim",
    opts = {
      colorscheme = "catppuccin",
    },
  },
}
```

Or select any other theme you like.
