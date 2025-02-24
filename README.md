# Pi NUT

Raspberry Pi NUT ([Network UPS Tools](https://networkupstools.org)) configuration for UPS monitoring and safe server shutdowns.

## Hardware Setup

<p align="center"><img alt="Pi NUT in Rack" src="/resources/nut-pi-rack.jpg" height="auto" width="600"></p>

Because UPSes generally have a single USB or serial port, and the Ethernet port (if your UPS has one) either requires a subscription to activate, or runs wildly outdated software (so it would be a risk to connect it to your network), NUT is useful in allowing a single computer (in my case, a Raspberry Pi) to share the UPS status information with other servers powered by it.

This playbook assumes you're running a Raspberry Pi like I am, connected directly to your UPS via USB, but you could run any computer with Debian or Ubuntu, including a little mini PC or some other SBC.

It's useful to run NUT on a low-power computer though, as it will be the last system to power down, and if it's a server burning 500W at idle, your UPS will likely be on its last legs as your NUT server shuts down!

Anyway, here's my hardware setup:

  - Raspberry Pi CM4 Lite with 1GB of RAM (used because I had one laying on my desk...)
  - [BigTreeTech Pi4B carrier board](https://amzn.to/4gIYGvX)
  - [3D printed Raspberry Pi Rackmount - right hand mount](https://www.printables.com/model/843677-raspberry-pi-5-rack-mount-right-sided) (I printed in ASA)
  - USB-C power supply, USB-A to USB-B cable for UPS, and Ethernet connected to my switch

## Known issues

  - This project is currently only configured for one computer (a Raspberry Pi) monitoring one UPS.
  - The NUT client configuration is only tested on Ubuntu and Debian Linux currently. Other Linux distributions may require changes.

## Software Setup

This project uses Ansible (`pip install ansible`).

The Ansible playbook depends on the `geerlingguy.nut_client` role, which can be installed with:

```
ansible-galaxy install geerlingguy.nut_client
```

All the servers to be managed by this playbook are listed inside `hosts.yml`. There should be one `primary` server, and then multiple `clients` which will subscribe to the primary NUT server for UPS status.

### Ansible Vault

This playbook uses Ansible Vault-encrypted secrets inside `config-secrets.yml` for the `nut_upsd_users` and `nut_client_password` variables.

The `ansible.cfg` file defines the path to my `vault_password_file`—but if you have that on _your_ machine... I'm a little worried!

You can delete my `config-secrets.yml` file and create your own with the variables you'd like to encrypt, then use `ansible-vault encrypt` to encrypt your version of the file.

I may rework the way the variable files are stored in this repository to make it easier for people to maintain their own forks, but right now it is the way it is :)

### Run the Ansible Playbook

To set up NUT on all configured hosts, make sure all the variables are configured correctly inside `config.yml` and `config-secrets.yml`, then run:

```
ansible-playbook main.yml
```

## Monitoring and Debugging

Run `upsc` to verify the UPS is being seen correctly:

```
$ upsc server-room-rack
Init SSL without certificate database
battery.charge: 24
battery.energysave: no
battery.packs: 1
battery.protection: yes
battery.runtime: 0
battery.voltage: 50.60
battery.voltage.nominal: 48.0
device.model: LILVX2K0
device.type: ups
driver.name: nutdrv_qx
...
```

You can monitor NUT's logs with:

```
# On server
journalctl -f -u nut-server

# On client nodes
journalctl -f -u nut-monitor
```

You can also manage the attached UPS directly with `upscmd`:

```
# List commands supported on this UPS
upscmd -l server-room-rack

# Run a quick battery test (requires password)
upscmd -u admin server-room-rack test.battery.start.quick
```

To perform a full test of NUT's shutdown without unplugging your UPS and draining the battery (NOTE: This will shut down all servers monitoring your UPS!), run:

```
upsmon -c fsd
```

Finally, there are a number of open source web UIs you can run to see a full status dashboard for your UPSes. I personally use [Home Assistant's NUT Integration](https://www.home-assistant.io/integrations/nut/) to display my UPS info on my Home Assistant power monitoring dashboard.

But if you don't have Home Assistant, the easiest Web UI to get started with is [nut_webgui](https://github.com/SuperioOne/nut_webgui). If you have Docker installed on your computer (this web UI doesn't _have_ to run on the NUT server), you can launch it with:

```
docker run \
  -e UPSD_ADDR=10.0.2.10 \
  -e UPSD_USER=observer \
  -e UPSD_PASS=PASSWORD_HERE \
  -p 9000:9000 \
  ghcr.io/superioone/nut_webgui:latest
```

Then visit `http://localhost:9000/` in your browser:

<p align="center"><img alt="NUT Web Monitor running in Docker" src="/resources/nut-web-monitor-docker.png" height="auto" width="600"></p>

## License

GPLv3

## Author Information

This project was created in 2025 by [Jeff Geerling](https://www.jeffgeerling.com/).
