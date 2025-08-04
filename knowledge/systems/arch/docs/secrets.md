<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Secrets

## Setup

Install `gnome-keyring`.

Enable and start daemon.

```shell
systemctl --user enable --now gnome-keyring-daemon
```

## Configuration

add to ~/.vscode/argv.json

```json
// use gnome keyring to store secrets
"password-store": "gnome-libsecret"
```
