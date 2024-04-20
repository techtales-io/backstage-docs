<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Setup Nvidia

Install drivers.

```shell
yay -Sy nvidia-open
```

Blacklist nouveau.

> vim /etc/modprobe.d/nvidia.conf

```shell
blacklist nouveau
blacklist nouveaufb
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias nouveaufb off
alias lbm-nouveau off
```
