<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Pipewire

## Setup

Install

```shell
yay -Sy \
  pipewire \
  pipewire-alsa \
  pipewire-pulse \
  pipewire-jack \
  pavucontrol-qt \
  qt6-wayland
```

Enable and start.

```shell
systemctl --user enable --now pipewire.socket
systemctl --user enable --now pipewire-pulse.socket
systemctl --user enable --now wireplumber.service
```
