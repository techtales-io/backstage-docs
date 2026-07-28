<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# AI Assistant Setup

## Main user bootstraps the assistant user

### Create user

disable password login

```shell
sudo useradd -m -g users -G docker -s /bin/bash assistant
sudo passwd -l assistant
```

spawn user manager at bootup that starts systemd processes for the assistant

```shell
sudo loginctl enable-linger assistant
```

### Setup ssh

create keypair

```shell
ssh-keygen
```

add the assistant host to `~/.ssh/config`

```shell
Host assistant
  HostName localhost
  User assistant
  IdentityFile ~/.ssh/id_ed25519
```

#### Setup ssh with yubikey touch

If you have a yubikey and want to press it every time you establish an ssh connection, then create a touch key pair.
VScode requires 2 touches per remote connection (anytime you open a remote folder).

```shell
ssh-keygen -t ed25519-sk
```

```shell
Host assistant
  HostName localhost
  User assistant
  IdentityFile ~/.ssh/id_ed25519_sk
```

### Setup yubikey access for the assistant

The yubikey needs to be present for the git signing.

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

### Setup shared space

#### Create the wiki dir

setgid (2) keeps dirs in users group

```shell
sudo mkdir -p /srv/shared/documents
sudo chown root:users /srv/shared/documents
sudo chmod 2775 /srv/shared/documents
```

assistant owns this wiki.

#### Create assistant dir and symlinks for read access

```shell
sudo mkdir -p /srv/shared/assistant
sudo mkdir -p /srv/shared/assistant/{context,plans,specs}
sudo chown -R assistant:users /srv/shared/assistant
sudo chown root:users /srv/shared/assistant
sudo chmod 2775 /srv/shared/assistant
ln -s /srv/shared/assistant ~/notes/
```

If you have syncthing, you can setup it now and wait for the files to sync.
if you need to adjust the permissions anytime:

```shell
sudo find /srv/shared/assistant -type d -exec chmod 2775 {} +
sudo find /srv/shared/assistant -type f -exec chmod 0664 {} +
```

## Switch to the assistant

The following is done as assistant user (via login/ssh).

```shell
sudo -u assistant /bin/bash
```

copy public key to ~/.ssh/authorized_keys
logout and ssh to the assistant.

```shell
ssh assistant
```

if you have dotfiles for the assistant, you can check them out now and run setup.

### Setup Git signing

```shell
systemctl --user enable --now ssh-agent
```

generate ssh key pair for signing only.

if you want to use yubikey, generate a fido key pair with no touch (edit file name and add "notouch", see below)

```shell
ssh-keygen -t -O no-touch-required ed25519-sk
```

set proper permissions

```shell
chmod 700 /home/assistant/.ssh
chmod 600 /home/assistant/.ssh/id_ed25519_sk_notouch
chmod 644 /home/assistant/.ssh/id_ed25519_sk_notouch.pub
```

add the key to the github signing keys page.

then generate ssh config

```shell
echo "AddKeysToAgent yes" > /home/assistant/.ssh/config
chmod 600 /home/assistant/.ssh/config
```

and add key to `~/.config/git/allowed_signers`

```shell
<user> <id.pub>
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

then configure a `mise.toml` in `.config/mise/config.toml`  for the default config
and one in any project folder that needs a different token, f.e. `projects/org`

```shell
[env]
_.file = [
    { path = "~/.secrets/github-token-org", redact = true },
]
```

don't forget to adjust your assistant's permissions so that it cannot read files from the secrets folder.
