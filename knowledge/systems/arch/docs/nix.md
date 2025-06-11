<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Nix

## Setup

```shell
pacman -Sy nix zsh-nix-shell
systemctl enable --now nix-daemon
```

Add user to nix group.

```shell
usermod -a -G nix-users <username>
```

## Configuration

Configure channels.

```shell
nix-channel --add https://nixos.org/channels/nixpkgs-unstable
nix-channel --update
```

## Usage

Run nix if shell.nix is present.

```shell
nix-shell
```

Run with explicit packages.

```shell
nix-shell -p git
```

Pin packages to create a reproducible environment.

```shell
nix-shell -p git --pure -I nixpkgs=<https://github.com/NixOS/nixpkgs/archive/2a601aafdc5605a5133a2ca506a34a3a73377247.tar.gz>
```
