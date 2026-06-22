# Fedora i3 Setup Notes

Personal setup guides for running **Fedora** with the **i3 window manager** — covering everything from a fresh install to GPU passthrough and daily tooling.

## Contents

| File | Description |
|------|-------------|
| [install.md](install.md) | Base system setup: system update, ASUS laptop tools (asusctl), NVIDIA drivers, SecureBoot module signing, display scaling, multi-monitor, fish shell, LazyGit, GPU switching (supergfxctl) |
| [i3.md](i3.md) | i3 tweaks: Picom compositor, feh wallpaper, borders, Catppuccin Mocha theme for i3bar and client windows |
| [i3-status.md](i3-status.md) | i3status bar configuration |
| [alacritty.md](alacritty.md) | Alacritty terminal config with Catppuccin Mocha theme, Iosevka Nerd Font, blur/opacity |
| [lazyvim.md](lazyvim.md) | LazyVim (Neovim) setup: install, Catppuccin theme, font options, file manager config, completion keybinds, line-moving shortcuts |
| [c#-setup.md](c#-setup.md) | .NET / C# development in LazyVim |
| [font.md](font.md) | Installing custom fonts |
| [fish-wifi.md](fish-wifi.md) | Fish shell WiFi helper function for i3 |
| [dark-system-themes.md](dark-system-themes.md) | System-wide dark theme for GTK3, GTK4, Qt5/Qt6 apps on X11 |
| [cups.md](cups.md) | Printer setup via CUPS |
| [qemu_kvm_vfio.md](qemu_kvm_vfio.md) | QEMU/KVM with VFIO GPU passthrough |

## Theme

All configs follow the **Catppuccin Mocha** color scheme consistently across i3, Alacritty, and Neovim.

## Hardware Context

These notes were written for an ASUS laptop with an NVIDIA dGPU and an external GPU (eGPU) option, running Fedora on X11 with i3.
