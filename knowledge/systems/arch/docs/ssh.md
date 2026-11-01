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

Enable and start.

```shell
sudo systemctl enable --now sshd.service
```

## Setup keys

### Fido2

Fido2 works with the default ssh-agent but its not needed if explicitly set in the ~/.ssh/config as `IdentityFile`.

Install

```shell
libfido2
```

Create keypair with touch.

```shell
ssh-keygen -t ed25519-sk
```

Create keypair with no touch.

```shell
ssh-keygen -O no-touch-required -t ed25519-sk
```

In `authorized_keys` on the host this pub key needs `no-touch-required sk-` in line before the key otherwise it will be rejected.

### TPM

Install

```shell
ssh-tpm-agent
tpm2-tools
tpm2-tss
```

Add yourself to the tss group.

```shell
sudo usermod -aG tss <user>
```

Reboot.

Create a tpm key - no passphrase.

```shell
ssh-tpm-keygen -t ecdsa -b 256 -f ~/.ssh/id_tpm
```

Add env variable to f.e. ~/.zshenv:

```shell
export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-tpm-agent.sock"
```

Enable and start ssh tpm agent service (part of `ssh-tpm-agent`).

```shell
systemctl --user enable --now ssh-tpm-agent.service
```

Verify.

```shell
ssh-add -L
```

Non-tpm keys need a fallback to the old socket.

Add to `~/.config/systemd/user/ssh-tpm-agent.service.d/override.conf`

```shell
[Service]
ExecStart=
ExecStart=/usr/bin/ssh-tpm-agent -A %t/ssh-agent.socket
```
