# Hyprland Setup Context

> Paste this at the top of any new Claude conversation to restore context.

## System

- **Distro:** OpenSUSE Tumbleweed
- **Goal:** Minimal Hyprland setup, initially tested in Oracle VirtualBox VM, target is bare metal dual boot
- **Display manager:** SDDM (replaced XDM which was the broken default)
- **Prior DE:** KDE Plasma (installed via KDE Plasma installer option)
- **Shell:** zsh

## Current Status

- SDDM installed and enabled, Hyprland appears as a session option at login
- **Hyprland black screens on launch — unresolved**
  - No log file created at `~/.local/share/hyprland/hyprland.log` — crashes before init
  - Suspected cause: VirtualBox virtual GPU can't support a GPU-accelerated Wayland compositor
  - Not yet tested on bare metal — that is the next real test
  - `journalctl -b 0 | grep -i hyprland` is the diagnostic to run on next attempt
  - GPU: not yet confirmed (`lspci | grep -E "VGA|3D|Display"`)

## Package Install

```bash
sudo zypper install hyprland sddm kitty \
  xdg-desktop-portal-hyprland \
  pipewire wireplumber \
  polkit polkit-kde-agent \
  hyprpaper waybar wofi \
  mako wl-clipboard grim slurp \
  hypridle hyprlock xwayland
```

## System Config Changes

**Enable SDDM:**
```bash
sudo systemctl enable --now sddm
```

**Verify Hyprland session file:**
```bash
ls /usr/share/wayland-sessions/
# hyprland.desktop should be present
```

**If hyprland.desktop is missing**, create it:
```ini
# /usr/share/wayland-sessions/hyprland.desktop
[Desktop Entry]
Name=Hyprland
Comment=An intelligent dynamic tiling Wayland compositor
Exec=Hyprland
Type=Application
```

**SDDM default session** is remembered automatically from last login. To set explicitly:
```ini
# /var/lib/sddm/state.conf
Session=hyprland
```

**TTY launch fallback** (add to `~/.zprofile` if not using SDDM):
```bash
if [ -z "$WAYLAND_DISPLAY" ] && [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
    exec Hyprland
fi
```

## Dotfiles / Git

- Tracking configs in a git repo (bare repo or stow — TBD)
- Configs to track from day one:
  - `~/.config/hypr/`
  - `~/.config/waybar/`
  - `~/.config/mako/`
  - `~/.config/wofi/`
  - `~/.config/hyprpaper/`
  - `~/.zprofile`

## Key Concepts Established

- Hyprland is a bare compositor — no wallpaper, bar, launcher, clipboard, or notifications out of the box
- Kitty is not a prerequisite; it's just the default terminal in `hyprland.conf` — swap it freely
- `xdg-desktop-portal-hyprland` is required for screen sharing and Flatpak file pickers
- XDM (X Display Manager) is X11-only and legacy — it will never show Wayland sessions
- SDDM reads `/usr/share/wayland-sessions/` for Wayland session options
- Hyprland must own the display — don't try to launch it inside an existing X11/Wayland session

## Next Steps

1. Test on bare metal after dual boot — VM is not a reliable proxy for Hyprland
2. Confirm GPU with `lspci | grep -E "VGA|3D|Display"`
3. If black screen persists on bare metal, check `journalctl -b 0 | grep -i hyprland` and `~/.local/share/hyprland/hyprland.log`
4. Once booting: configure `~/.config/hypr/hyprland.conf` — keybindings, monitor layout, autostart
5. Wire up waybar, mako, hyprpaper, wofi as autostart entries in hyprland.conf
6. Initialize dotfiles git repo
## Reference

- Hyprland wiki: https://wiki.hyprland.org — read Getting Started and Configuring sections systematically
