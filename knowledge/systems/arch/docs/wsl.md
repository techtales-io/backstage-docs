<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# WSL2 with Arch custom image

## Create medium

in a linux environment

```shell
tar -zxvf archlinux-bootstrap-<version>-x86_64.tar.gz
tar -zcvf archlinux-bootstrap-<version>-x86_64_WSL.tar.gz root.x86_64/
```

## Setup WSL2

install from store, if not yet done.

```shell
https://aka.ms/wslstorepage
```

## Import to WSL2

create dir in user home on windows environment, import arch to wsl and start.

```shell
mkdir ./.wsl
wsl --import arch .\.wsl\arch .\Downloads\archlinux-bootstrap-<version>-x86_64_WSL.tar.gz
wsl -d arch
```

## Setup arch

initialize pacman

```shell
pacman-key --init
pacman-key --populate archlinux
```

update mirrorlist

```shell
curl "https://archlinux.org/mirrorlist/?country=AT&protocol=https&ip_version=4&use_mirror_status=on" > /etc/pacman.d/mirrorlist.new
sed "s/#Server/Server/" /etc/pacman.d/mirrorlist.new > /etc/pacman.d/mirrorlist
rm /etc/pacman.d/mirrorlist.new
```

install base

```shell
pacman -Syu
pacman -S base base-devel git vim
```

## Configure WSL2

create wsl config

```shell
vim /etc/wsl.conf
```

```shell
[boot]
systemd=true
[network]
hostname=wsl
generateHosts=false
[user]
default=<username>
```

## Configure base system

Continue with configuring [base system](installation.md#configure-base-system).

## Finish installation

```shell
wsl —terminate arch
wsl -s arch # sets as default
```

## SSH

To use ssh agent, install keychain.

```shell
yay -Sy keychain
```

Add this to .zshrc (or some else .zsh file, that gets sourced). Reference the key.

```shell
if command -v keychain > /dev/null 2>&1; then
  eval $(keychain -q --nogui --eval id_ed25519)
fi
```
