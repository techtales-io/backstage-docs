<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Reset Arch

## Cleanup packages

Change install reason from `explicit` to `dependency`.

```shell
pacman -D --asdeps $(pacman -Qqe)
```

Set packages that should be kept as explicit.

```shell
pacman -D --asexplicit \
  base \
  base-devel \
  intel-ucode \
  linux \
  linux-firmware
```

Delete not needed packages.

```shell
pacman -Qtdq | sudo pacman -Rns -
```
