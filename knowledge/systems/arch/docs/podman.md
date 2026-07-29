<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Podman (rootless)

## Setup

```shell
pacman -Sy podman podman-compose
```

## Configuration

.config/containers/containers.conf

```shell
[containers]
pids_limit = -1
userns = "auto"

[network]
pasta_options = ["--ipv4-only"]
```

.config/containers/registries.conf

```shell
unqualified-search-registries = ["docker.io"]
```

For podman to work, export this to your shell if not yet done:

```shell
export XDG_RUNTIME_DIR=/run/user/$(id -u)
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
export PATH="$HOME/.local/bin:$PATH"
```

Now create symlink to your home, so callers find docker there which is podman.

```shell
ln -sf /usr/bin/podman ~/.local/bin/docker
```

Enable and start.

```shell
systemctl --user enable --now podman.socket
```
