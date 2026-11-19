<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# AI Assistant Setup

## Bootstrap assistant user and configure

Main user does the assistant user setup.

### Create user

disable password login

```shell
sudo useradd -m -g users -s /bin/bash assistant
sudo passwd -l assistant
```

spawn user manager at bootup that starts systemd processes for the assistant

```shell
sudo loginctl enable-linger assistant
```

### Setup ssh

create keypair (optionally with fido2 or tpm, see [ssh.md](./ssh.md))

add the assistant host to `~/.ssh/config`

```shell
Host assistant
  HostName localhost
  User assistant
  IdentityFile ~/.ssh/id_xxx
```

### Fido2 access

Non-root user need access to yubikey.

check if key exists, then get vendor id

```shell
lsusb
lsusb -v | grep Yubi -B 10 -A 10
```

set udev rule for notouch signing with yubikey on github commits (remove 0x)

`/etc/udev/rules.d/70-yubikey.rules`

```shell
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1050", MODE="0660", GROUP="users"
```

then restart! an udev trigger is not enough.

## Setup Assistant

The following is done as assistant user (via login/ssh).

```shell
sudo -u assistant /bin/bash
```

copy public key to ~/.ssh/authorized_keys
If you have a no touch key, add before the public key `no-touch-required sk-ssh...` otherwise the key will get rejected.

logout and ssh to the assistant.

```shell
ssh assistant
```

### Setup Home

if you have dotfiles for the assistant, you can check them out now and run setup.

if you have syncthing, you can setup a ~/notes folder for the assistant.
this folder can be synced to other homes as well, simply enable syncthing service for the user.
See [syncthing.md](./syncthing.md)

### Setup Git signing

setup ssh (optionally with fido2 no-touch or tpm, see [ssh.md](./ssh.md))

add the key to the github signing keys page.

then generate ssh config

```shell
echo "AddKeysToAgent yes" > ~/.ssh/config
chmod 600 ~/.ssh/config
```

and add key to `~/.config/git/allowed_signers`

```shell
<github user> <id.pub>
```

setup git config in `~/.config/git/config`

### Secrets and auth

you can use the gh cli with oauth, if you do not have elevated privileges.
otherwise its recommended to provide fine grained tokens to the assistant.

add tokens to ~/.secrets/

```shell
echo "PERSONAL_TOKEN" > ~/.secrets/github-token-personal
echo "ORG_TOKEN" > ~/.secrets/github-token-org
```

then configure a `mise.toml` in `.config/mise/config.toml` for the default config
and one in any project folder that needs a different token, f.e. `projects/org`

```shell
[env]
_.file = [
    { path = "~/.secrets/github-token-org", redact = true },
]
```

don't forget to adjust your assistant's permissions so that it cannot read files from the secrets folder.

## Setup Openchamber

### Setup as assistant user

install Openchamber and Opencode with bun to keep update functionality.

```shell
bun add -g @openchamber/web
bun add -g --trust @opencode/cli
```

create systemd unit for the server `openchamber.service`

```shell
[Unit]
Description=OpenChamber webserver
After=network-online.target

[Service]
Type=simple

Environment="PATH=%h/.bun/bin:/usr/local/bin:/usr/bin:/bin"
EnvironmentFile=-%h/.config/openchamber/startup.env
ExecStart=/usr/bin/bun %h/.bun/bin/openchamber serve --foreground --port 3000 --host 0.0.0.0 --api-only

Restart=always
RestartSec=5
SuccessExitStatus=143

[Install]
WantedBy=default.target
```

link it

```shell
systemctl --user link openchamber.service
```

set password for web ui

```shell
mkdir -p ~/.config/openchamber
echo "OPENCHAMBER_UI_PASSWORD=<pass>" > ~/.config/openchamber/startup.env
```

enable and start

```shell
systemctl enable --now --user openchamber.service
```

verify

```shell
systemctl status --user openchamber.service
```

### Setup as main user

download and install app from `https://openchamber.dev/download/`.

#### desktop app

```shell
mkdir -p ~/.local/bin
install -m 755 OpenChamber-*-linux-x86_64.AppImage ~/.local/bin/openchamber
```

in the desktop app on the same machine goto settings > remote instances > other openchamber servers.
add server and click save.

```shell
http://127.0.0.1:3000
```

on connect (left bottom corner `Local`) password will be asked once > trust the device.

#### phone

on the same machine:

in the desktop app goto settings > remote instances > connect to this server > add device.
scan the qr code with your phone.

on another machine:

```shell
openchamber connect-url --port 3000 --qr --name hostname
```

## Setup Hermes

### Setup as assistant user

install hermes on the assistant (headless).

```shell
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash -s -- --non-interactive --skip-browser --skip-computer-use
```

create systemd unit for the backend `hermes.service`

```shell
[Unit]
Description=Hermes Agent backend
After=network-online.target
Wants=network-online.target

[Service]
WorkingDirectory=%h/.hermes

Environment=PATH=%h/.local/share/mise/shims:%h/.local/bin:/usr/local/bin:/usr/bin:/bin
ExecStart=%h/.local/bin/hermes serve --host 0.0.0.0 --port 9119 --skip-build

Restart=always
RestartSec=5
RestartPreventExitStatus=78

[Install]
WantedBy=default.target
```

link it

```shell
systemctl --user link hermes.service
```

next configure the hermes backend.

the password hash needs to be encrypted with scrypt.

```shell
cd ~/.hermes/hermes-agent && \
PYTHONPATH="$PWD" ~/.hermes/installs/*/environments/*/venv/bin/python -c \
'import getpass; from plugins.dashboard_auth.basic import hash_password; print(hash_password(getpass.getpass("Password: ")))'
```

add to `~/.hermes/.env`

```shell
# Dashboard login
HERMES_DASHBOARD_BASIC_AUTH_USERNAME=<your user>
HERMES_DASHBOARD_BASIC_AUTH_PASSWORD_HASH='scrypt$...'
HERMES_DASHBOARD_BASIC_AUTH_SECRET=<output of: openssl rand -base64 32>
```

protect the .env

```shell
chmod 600 ~/.hermes/.env
```

enable and start

```shell
systemctl enable --now --user hermes.service
```

verify

```shell
systemctl status --user hermes.service
```

optional setup rtk for hermes. rtk token saver needs to be in path.

```shell
rtk init --agent hermes
```

### Setup as main user

install hermes on the main user.

```shell
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash -s -- --non-interactive --skip-browser --skip-computer-use --include-desktop
```

#### desktop app

add a menu entry for hermes-desktop to `~/.local/bin/`.

```shell
#!/usr/bin/env bash
exec hermes desktop "$@"
```

in the desktop app goto settings > gateways > primary gateway.
add server, auth to it, enable encrypt saved secrets with the OS keychain and click save.

on the same machine:

```shell
http://127.0.0.1:9119
```
