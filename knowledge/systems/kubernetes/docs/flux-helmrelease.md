<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD046 -->

# Flux helmrelease howtos

## Howto fix stuck helmrelease

```shell
Helm upgrade failed: another operation (install/upgrade/rollback) is in progress
```

check for status of stuck app

```shell
helm status -n example example-app
```

unfortunately its stuck in pending-upgrade

```shell
NAME: example-app
LAST DEPLOYED: Wed Nov 22 12:56:04 2023
NAMESPACE: example
STATUS: pending-upgrade
REVISION: 4
TEST SUITE: None
```

do a rollback to the previous one:

```shell
helm rollback example-app 3 -n example
```

it should count one up and change the status to `deployed`.

```shell
NAME: example-app
LAST DEPLOYED: Wed Nov 22 20:39:27 2023
NAMESPACE: example
STATUS: deployed
REVISION: 5
TEST SUITE: None
```

at next reconcile it should upgrade to the latest version if possible.
