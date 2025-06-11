<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# SSH

## Setup

Install `gnome-keyring`.

Enable and start daemon and ssh agent.

```shell
systemctl --user enable --now gnome-keyring-daemon
systemctl --user enable --now gcr-ssh-agent.socket
```

### Configuration

Add to startup (f.e. .zshrc, .xinitrc)

```shell
dbus-update-activation-environment --all
export SSH_AUTH_SOCK=/run/user/1000/gcr/ssh
```
