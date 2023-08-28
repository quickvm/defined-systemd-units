# defined-systemd-units

This is a small collection of systemd units and supporting scripts to help you install and enroll [dnclient](https://docs.defined.net/glossary/dnclient/) into a [defined.net](https://defined.net/) account on a server that uses systemd.

## Requirements

* A free [defined.net](https://admin.defined.net/signup) account
* A configured [overlay network](https://docs.defined.net/guides/getting-started/)
* [Bash 4.2+](https://www.gnu.org/software/bash/)
* [systemd](https://systemd.io/)
* [envsubst](https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html)
* [jq](https://stedolan.github.io/jq/)
* [curl](https://curl.se/)
* The need for more overlay networking in your life so you can show off to your friends

## Install

1. Clone (or download) this repository `git clone https://github.com/quickvm/defined-systemd-units.git`
1. `cd defined-systemd-units`
1. `sudo ./install`

If you want to customize the install you can export the following environment variables:

```bash
export UNIT_FILES=/full/path/to/defined-systemd-units/units
export UNIT_DIR=/usr/lib/systemd/system
export BIN_FILES=/full/path/to/defined-systemd-units/bin
export BIN_DIR=/usr/bin
export DN_VERSION=1.2.1
```

Note: The `dnclient.service` unit that this repository provides expects `dnclient` to support systemd type=notify. The current `v0.2.2-beta3` version of `dnclient` has this support, so the `install` script overrides the version to `v0.2.2-beta3`. Setting `DN_VERSION` to a lower version might not work as expected. You can also change `Type=notify` in `units/dnclient.service` to `Type=simple` and use an older version if you wish.

Note: The `install` script uses `envsubst` to template the systemd units in the `units` directory. If you copy them directly they will not work! Please make your changes as you see fit.

## Configuration

You will need to collect a few bits of information from [defined.net](https://admin.defined.net/). You will need:

1. An [API key](https://docs.defined.net/guides/automating-host-creation/#creating-an-api-key) with the following scopes:
    * hosts:create
    * hosts:delete
    * hosts:enroll
1. A Network ID. Your `networkID`. It can be found in the top left section of the menu in the admin panel.
1. A [Role](https://docs.defined.net/guides/creating-firewalls-using-roles/#creating-roles) ID

Once you have the above information you will want to save these as key value pairs in `/etc/defined/dnctl`.

```bash
DN_API_key=dnkey-AEMEIG2EITHATEI8WEEL1UBEE2-EI3WUOVEEN3AHV9OV1OHROO8ZEI3GESHIE2ICH3JI4FIQUOH5FUO # Required. The API key used to enroll the host.
DN_NETWORK_ID=network-MAI8WU2YAHN5THEESOOKEE7NAH # Required. The network that the host will enroll into.
DN_ROLE_ID=role-AICHING2OHGHEI9QUEIZOO7EIZ # Required. The role that the host will enroll into.
DN_SKIP_UNENROLL=true # Optional. If set to true the host will not unenroll on reboot or shut down. Defaults to false.
DN_IP_ADDRESS=100.100.0.10 # Optional. If set to an IP in your defined.net network CIDR range it will enroll the host with that IP address.
DN_NAME="my-custom-name-$(hostname)" # Optional. You can customize the name you give your host in defined.net. Defaults to dsu-$(hostname)
```

Note: `DN_SKIP_UNENROLL=true` should only be used on hosts that you want to stay enrolled over a long period of time. If you set `DN_SKIP_UNENROLL` on servers that are ephemeral in nature you will end up with a bunch of enrolled hosts that you will manually have to clean up at some point. Set `DN_SKIP_UNENROLL=true` with care!

Optionally you can export them to your shell environment as the `root` user:

```bash
export DN_API_key=dnkey-AEMEIG2EITHATEI8WEEL1UBEE2-EI3WUOVEEN3AHV9OV1OHROO8ZEI3GESHIE2ICH3JI4FIQUOH5FUO
export DN_NETWORK_ID=network-MAI8WU2YAHN5THEESOOKEE7NAH
export DN_ROLE_ID=role-AICHING2OHGHEI9QUEIZOO7EIZ
```

and then as the `root` user run `dnctl write_config` or use `sudo -E` and it will create `/etc/defined/dnctl` for you.

This is useful if you are enrolling an ephemeral environment such as a GitHub Action. Check out our GitHub Action [action-dnclient](https://github.com/quickvm/action-dnclient) which can be used to join your Defined.net overlay network. Think deployments on internal infra or accessing internal resources from your GitHub Action runs.

Note: If you export the above to your shell environment you need to `sudo su -` to the `root` user first or use `sudo -E` to pass the current environment through sudo. If you run `sudo -E dnctl write_config` as a non root user, the exported `DN_` variables might not work. When in doubt, `sudo su -` to the `root` user, export your `DN_` vars and run `dnctl write_config`.

## Enable and start `dnclient.service` and `dnctl.service`

You should be ready to enable and start `dnclient.service`. You can either do this via `dnctl`:

1. `dnctl enable`
1. `dnctl start`

or via `systemctl`:

1. `systemctl enable dnclient.service dnctl.service`
1. `systemctl start dnclient.service`

Note: You only need to start `dnclient.service` since it has a systemd.unit `Wants=dnctl.service` directive it will cause `dnctl.service` to start afterwards. This dependency will cause dnctl.service to stop if dnclient.service stops. If `DN_SKIP_UNENROLL=true` is set, this will prevent the host from being unenrolled by `dnctl.service` when it stops.

Once the units are enabled and started you can check out <https://admin.defined.net/hosts> and you should see your host enrolled automatically! Tada!

## License

MIT License

Copyright (c) 2023 QuickVM

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
