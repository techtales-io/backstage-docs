<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Docker

## Setup

```shell
pacman -Sy docker
```

Enable and start docker.

```shell
systemctl enable --now docker
```

Add user to docker group.

```shell
usermod -a -G docker <username>
```
