<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Syncthing

## Setup

Install `syncthing`.

Enable and start daemon.

```shell
systemctl --user enable --now syncthing
```

## Configuration

Goto <http://localhost:8384>
Goto <https://syncthing.techtales.io>

First on localhost: add remote device main with ID
Then on remote: add folder by folder and map to local path
