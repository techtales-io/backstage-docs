<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# SSH

## Enable Agent

Enable and start ssh agent service (part of `openssh`).

```shell
systemctl --user enable --now ssh-agent.service
```

Add env variable to f.e. ~/.zshenv:

```shell
export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"
```

## Enable Daemon

create config

```shell
sudo vim /etc/ssh/sshd_config.d/20-force_publickey_auth.conf
```

```shell
AuthenticationMethods publickey
PasswordAuthentication no
PermitRootLogin no
```

### Systemd

```shell
sudo systemctl enable --now sshd.service
```
