# How to set up system theme to dark in applications

How to setup system-wide dark theme support for GTK3, GTK4, and Qt5/Qt6 applications on i3 (X11), using the fish shell.

## Install packages

```bash
sudo dnf install xsettingsd qt5ct qt6ct adw-gtk3-theme
```

## Configure GTK3 and GTK4

Create the settings files for both GTK versions:
```bash
mkdir -p ~/.config/gtk-3.0 ~/.config/gtk-4.0

echo "[Settings]
gtk-application-prefer-dark-theme=1
gtk-theme-name=adw-gtk3-dark
gtk-icon-theme-name=Adwaita
gtk-font-name=Iosevka Nerd Font Mono 11
gtk-cursor-theme-name=Adwaita" > ~/.config/gtk-3.0/settings.ini

echo "[Settings]
gtk-application-prefer-dark-theme=1
gtk-theme-name=adw-gtk3-dark
gtk-icon-theme-name=Adwaita
gtk-font-name=Iosevka Nerd Font Mono 11
gtk-cursor-theme-name=Adwaita" > ~/.config/gtk-4.0/settings.ini
```

## Configure xsettingsd

This daemon propagates the theme to running GTK applications dynamically.
```bash
mkdir -p ~/.config/xsettingsd

echo 'Net/ThemeName "adw-gtk3-dark"
Net/IconThemeName "Adwaita"
Gtk/FontName "Iosevka Nerd Font Mono 11"
Gtk/CursorThemeName "Adwaita"' > ~/.config/xsettingsd/xsettingsd.conf
```

And add it to your i3 config file in `~/.config/i3/config`:
```bash
exec_always --no-startup-id xsettingsd
```

## Configure Qt5/Qt6

Set platform theme environment variable in `~/.config/fish/config.fish`:
```bash
set -x QT_QPA_PLATFORMTHEME qt5ct
```

Set dark themes and fonts for Qt5 and Qt6 in configurators:
```bash
qt5ct
qt6ct
```

In each tool:
 1. Go to **Appearance** tab
 1. Select `Adwaita-dark` (or `Fusion` style with the `darker.conf` palette under the **Palette** tab)
 1. Save
