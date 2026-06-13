<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Podman

## Setup

```shell
pacman -Sy podman podman-docker podman-compose
```

Add subuid/subgrids to user.

```shell
sudo usermod --add-subuids 10000-75535 user
sudo usermod --add-subgids 10000-75535 user
```

## Configuration

Configure containers if migrating from docker.

.config/containers/registries.conf

```shell
[containers]
privileged = false
read_only = true
userns = "auto"
```

.config/containers/registries.conf

```shell
unqualified-search-registries = ["docker.io"]
```

## Usage

Run as usual with docker commands (podman-docker).
