# Juju/LXD tweaks

The purpose of this repo is to share the tweaks I've made to improve the performance of Juju when used with the local provider (LXD).

## Copy-on-write

## Disable automatic apt update and upgrades

### At bootstrap
```base
juju bootstrap lxd devel --config enable-os-refresh-update=false --config enable-os-upgrade=false
```
### After bootstrap

For the current model:
```bash
juju model-config enable-os-refresh-update=false enable-os-upgrade=false
```

For all new models:
```bash
juju model-defaults enable-os-refresh-update=false enable-os-upgrade=false
```

## update-juju-lxd-images

When Juju creates a new machine, it looks for an LXD image with the alias `juju/$series/amd64`. If no image is found, a new one will be downloaded.

Next, Juju's provisioner will download the Juju agent to the machine.

Finally, when the charm is deployed, it's pre-install will download apt and pip packages to support reactive charms.

This script aims to pre-cache most of that work, by building an image that has the Juju agent and basic reactive charm dependencies pre-installed.

At the time it was written, that included 111 packages and reduced the average time to deploy a new Juju unit from ~7 minutes to ~1 minute.

It does not attempt to pre-install pip packages, but that is the next logical step.

Currently, it only caches the image for the most recent Long Term Supported (LTS) release: Bionic. Previous releases are commented out if you're like to re-enable them.

To use, you can run it manually or via crontab.
