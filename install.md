# Update system
```bash
sudo dnf update -y
sudo reboot
```

# Terra repo + asusctl

ASUS Linux packages are available via Terra Repository.
```bash
sudo dnf install --nogpgcheck \
  --repofrompath 'terra,https://repos.fyralabs.com/terra$releasever' \
  terra-release
```

Install `asusctl` and run the service:
```bash
sudo dnf install asusctl
systemctl enable --now asusd.service
```

To avoid conflicts with tuned, change it to power-profiles-dameon:
```bash
sudo dnf swap tuned-ppd power-profiles-daemon --allowerasing
systemctl enable --now power-profiles-daemon.service
```

Optionall GUI
```bash
sudo dnf install asusctl-rog-gui
```

# NVIDIA Driver (dGPU + eGPU)

Add RPM Fusion and driver
```bash
sudo dnf install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda
```

After RPM finishes wait ~5 min - kernel module needs to be rebuild. Afterwards reboot:
```bash
sudo reboot
```

# Display scaling in i3

## Check output names

```bash
xrandr
```

You will see list of video outputs, e.g. `eDP-1` for laptop or `HDMI-1`, `DP-1` for external monitors.

## Set DPI globally via `X.resources`

Create/Edit `~/.Xresources`:
```bash
Xft.dpi: 144
Xft.autohint: 0
Xft.lcdfilter: lcddefault
Xft.hintstyle: hintfull
Xft.hinting: 1
Xft.antialias: 1
Xft.rgba: rgb
Xcursor.size: 32
```

DPI values: `96` = no scaling, `120` = 125%, `144` = 150%, `192` = 200%

## Load Xresources automatically with i3

In `~/.config/i3/config` add:
```bash
exec_always --no-startup-id xrdb -merge ~/.Xresources
```

# Disable mouse acceleration

Create file `/etc/X11/xorg.conf.d/50-mouse-acceleration.conf`:
```bash
Section "InputClass"
    Identifier "libinput pointer catchall"
    MatchIsPointer "on"
    MatchDevicePath "/dev/input/event*"
    Driver "libinput"
    Option "AccelProfile" "flat"
EndSection
```

Option `AccelProfile "flat` disables mouse acceleration and changes `libinput Accel Profile Enabled` in xinput to correct state.

# Optional: Cardwire (dynamic GPU changing without reboot)

Cardwire it's experimental alternative for supergfxd
```bash
sudo dnf install cardwire
```

# Signign and loading NVIDIA drivers

## Install tools

```bash
sudo dnf install kmodtool akmods mokutil openssl
```

## Generate MOK key (it might already be created)

```bash
sudo kmodgenca -a
```

Flag `-a` uses default values and generates the certificate automatically.
Private key lands in `/etc/pki/akmods/private/`, public certificate in to `/etc/pki/akmods/certs/public_key.der`.

Check if file exists:
```bash
ls /etc/pki/akmods/certs/public_key.der
```

## Enroll the key in UEFI

```bash
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
```

Program will ask for password, anything will work, something as easy as `1234` also will work. It's only needed *once* during key enrolment in MOK Manager.

## Reboot and enroll the key in MOK Manager

```bash
sudo reboot
```

After restart you will see *blue screen with MOK Management*
 1. Select *Enroll MOK*
 1. Select *Continue*
 1. Select *Yes*
 1. Input password set in previous step
 1. Select *OK* -> system will restart

## Force rebuild of module and initramfs

```bash
sudo akmods --force
sudo dracut --force
sudo reboot
```

## If modules still don't load then force the rebuild with signign

```bash
# Remove old build modules
sudo rm -rf /usr/lib/modules/$(uname -r)/extra/nvidia/

# Force full rebuild with signign
sudo akmods --force --rebuild

# Wait for it to finish (3-5 minutes), then check logs
sudo journalctl -u akmods -f
```

And then again:
```bash
sudo akmods --force --rebuild
sudo dracut --force
sudo reboot
```

After restart:
```bash
sudo modprobe nvidia && echo "OK" || echo "ERROR"
```

# Adding second monitor

In my setup:
```bash
xrandr --output HDMI-1-0 --mode 3440x1440 --rate 74.98 --primary \
       --output eDP --mode 1920x1200 --right-of HDMI-1-0
```

And in `~/.config/i3/config` add:
```bash
exec_always --no-startup-id xrandr --output HDMI-1-0 --mode 3440x1440 --rate 74.98 --primary --output eDP --mode 1920x1200 --right-of HDMI-1-0
```

Or even better do it with external script for easier adjustments:
```bash
# Create script
mkdir -p ~/.config/i3/scripts
cat > ~/.config/i3/scripts/displays.sh << 'EOF'
#!/bin/bash
xrandr --output HDMI-1-0 --mode 3440x1440 --rate 74.98 --primary \
       --output eDP --mode 1920x1200 --right-of HDMI-1-0
EOF
chmod +x ~/.config/i3/scripts/displays.sh
```

In `~/.config/i3/config`:
```bash
exec_always --no-startup-id ~/.config/i3/scripts/displays.sh
```

After editing the file reload i3 via `$mod+Shift+r` - script will run and set the screens.

# Install Alacritty and set it as default terminal in i3

## Install

```bash
sudo dnf install alacritty
```

In `~/.config/i3/config` find line with terminal (default `i3-sensible-terminal`) and change it to:
```bash
bindsym $mod+Return exec alacritty
```

Reload i3:
```bash
i3-msg restart
```

## Basic config

```bash
mkdir -p ~/.config/alacritty
cat > ~/.config/alacritty/alacritty.toml << 'EOF'
[window]
padding = { x = 8, y = 8 }
decorations = "none"

[font]
size = 11.0

[font.normal]
family = "monospace"

[cursor]
style = { shape = "Block", blinking = "On" }
EOF
```

# Install and use fish

## Install

```bash
sudo dnf install fish
```

## Set default shell to fish

```bash
chsh -s /usr/bin/fish
```

Logout and login to see the changes.

# Supergfctl if Asusctl or Cardwire fails to change GPUs

## Clone, build and install

```bash
# Install dependencies
sudo dnf install rust cargo git dbus-devel systemd-devel

# Clone repo
git clone https://gitlab.com/asus-linux/supergfxctl.git
cd supergfxctl

# Build and install
make
sudo make install

# Enable the service
sudo systemctl enable --now supergfxd.service

# Add yourself to the group
sudo usermod -aG users $USER
```

## Scripts for easy use in i3

Create the script file:
```bash
nano ~/.config/i3/scripts/gpu-switch.sh
```

And add:
```bash
#!/bin/bash

CURRENT=$(supergfxctl --get 2>/dev/null | tr -d '[:space:]')

show_status() {
    echo "Aktualny tryb: $CURRENT"
    echo "Dostępne tryby: $(supergfxctl --supported 2>/dev/null)"
}

switch_and_logout() {
    local mode=$1
    echo "Przełączam na: $mode"
    sudo supergfxctl --mode $mode
    sleep 1
    i3-msg exit
}

case "$1" in
    igpu|integrated)
        switch_and_logout Integrated
        ;;
    hybrid)
        switch_and_logout Hybrid
        ;;
    egpu)
        switch_and_logout AsusEgpu
        ;;
    status)
        show_status
        ;;
    *)
        echo "Użycie: gpu-switch [igpu|hybrid|egpu|status]"
        show_status
        ;;
esac```

```bash
chmod +x ~/.config/i3/scripts/gpu-switch.sh
```

Optionally add yourself to sudoers so it won't ask for password
```bash
sudo visudo
```

Add line:
```bash
your_user ALL=(ALL) NOPASSWD: /user/bin/supergfxctl
```

Optionally add shortcuts in i3
```bash
bindsym $mod+F1 exec --no-startup-id ~/.config/i3/scripts/gpu-switch.sh igpu
bindsym $mod+F2 exec --no-startup-id ~/.config/i3/scripts/gpu-switch.sh hybrid
bindsym $mod+F3 exec --no-startup-id ~/.config/i3/scripts/gpu-switch.sh egpu
```

Usage:
```bash
gpu-switch status    # current mode
gpu-switch igpu      # only integrated graphics, will logout
gpu-switch hybrid    # integrated + dGPU, will logout
gpu-switch egpu      # enable eGPU, will logout
```
