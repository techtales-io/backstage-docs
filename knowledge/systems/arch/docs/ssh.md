<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# SSH

## Setup

Enable and start ssh agent service (part of `openssh`).

```shell
systemctl --user enable --now ssh-agent.service
```

## Configuration

Add env variable to f.e. ~/.zshenv:

```shell
export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"
```
