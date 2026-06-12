# Install

## Install neovim

```bash
sudo dnf install neovim
```

## Install lazyvim dependencies

```bash
sudo dnf install git gcc make ripgrep fd-find
```

## Install LazyVim

```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
```

# Optional settings

## Change default colorscheme:

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

## Font and other options

Edit `~/.config/nvim/lua/config/options.lua`:
```lua
vim.opt.guifont = "Iosevka Nerd Font:h12"
vim.api.nvim_set_hl(0, "Normal", { bg = "none" })
vim.api.nvim_set_hl(0, "NormalFloat", { bg = "none" })
vim.api.nvim_set_hl(0, "NormalNC", { bg = "none" })
vim.api.nvim_set_hl(0, "SignColumn", { bg = "none" })
vim.api.nvim_set_hl(0, "LineNr", { bg = "none" })
```

Additionally for fonts settings edit `~/.config/nvim/lua/plugins/ui.lua`
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

## Selecting file manager

By default when opening `nvim` in a directory, e.g. `nvim .` or `nvim .config` etc. both `neo-tree` and `snacks` is open. You can disable either one and keep using only one file manager.

Edit `~/.config/nvim/lua/plugins/ui.lua`:

### Disable `snacks`

```lua
{
  "folke/snacks.nvim",
  opts = {
    explorer = { enabled = false },
    dashboard = { enabled = false }, -- opcjonalnie
  },
},
```

### Disable `neo-tree`

```lua
{
  "nvim-neo-tree/neo-tree.nvim",
  enabled = false,
},
```

## Change how auto completion works

If you don't like `enter` key accepting line completions and to use `tab` instead, then edit `~/.config/nvim/lua/plugins/cmp.lua`:
```lua
return {
  "hrsh7th/nvim-cmp",
  opts = function(_, opts)
    local cmp = require("cmp")
    opts.mapping = vim.tbl_extend("force", opts.mapping, {
      ["<CR>"] = cmp.mapping(function(fallback)
        fallback()
      end, { "i", "s" }),
      ["<Tab>"] = cmp.mapping(function(fallback)
        if cmp.visible() and cmp.get_active_entry() then
          cmp.confirm({ select = false })
        else
          fallback()
        end
      end, { "i", "s" }),
    })
    return opts
  end,
}
```

If you still want to accept suggestion with `enter` but also for `enter` to be able to create new line when suggestions are open but none is selected, change to:
```lua
return {
  "hrsh7th/nvim-cmp",
  opts = function(_, opts)
    local cmp = require("cmp")
    opts.mapping = vim.tbl_extend("force", opts.mapping, {
      ["<CR>"] = cmp.mapping(function(fallback)
        if cmp.visible() and cmp.get_active_entry() then
          cmp.confirm({ select = false })
        else
          fallback() -- normalny enter
        end
      end, { "i", "s" }),
    })
    return opts
  end,
}
```
