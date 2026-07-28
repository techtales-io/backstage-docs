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
userns = "auto"

[network]
pasta_options = ["--ipv4-only"]
```

.config/containers/registries.conf

```shell
unqualified-search-registries = ["docker.io"]
```

Enable and start.

```shell
systemctl --user enable --now podman.socket
```

Optional set `alias docker="podman"`.
