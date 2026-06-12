# C# development using LazyVim

## Install dotnet

```bash
# Check installed version
dotnet --info

# If dotnet is not installed, install it. E.g. SDK version 10.0
sudo dnf install dotnet-sdk-10.0
```

## Add LazyVim plugins
Edit `~/.config/nvim/lua/config/lazy.lua` and in section `spec` add:
```lua
{ import = "lazyvim.plugins.extras.lang.dotnet" },
```
