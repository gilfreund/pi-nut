# Pi NUT

Raspberry Pi NUT ([Network UPS Tools](https://networkupstools.org)) configuration for UPS monitoring and safe server shutdowns.

## Hardware Setup

TODO:

  - Raspberry Pi of practically any variety (I'm using CM4 Lite with 1GB of RAM on [BigTreeTech Pi4B carrier board](https://amzn.to/4gIYGvX).
  - Built for Debian, Ubuntu, or a derivative OS.

## Software Setup

This project uses Ansible (`pip install ansible`).

The playbook depends on the `geerlingguy.nut_client` role, which can be installed with:

```
ansible-galaxy install geerlingguy.nut_client
```

All the servers to be managed by this playbook are listed inside `hosts.yml`. There should be one `primary` server, and then multiple `clients` which will subscribe to the primary NUT server for UPS status.

To set up NUT on all configured hosts, run:

```
ansible-playbook main.yml
```

> **NOTE**: This playbook uses an Ansible Vault-encrypted secret inside `config-secrets.yml` for the `nut_client_password` variable. The `ansible.cfg` file defines the path to my `vault_password_file`—but if you have that on _your_ machine... I'm a little worried! You can either modify the playbook to remove the use of an encrypted secrets file, or you can use `ansible-vault encrypt` to encrypt your own secrets file with the client password inside and use _that_.

## Debugging

I will add more to this section later.

For now, see my blog post: [I have NUT on my Pi, so my servers don't die](https://www.jeffgeerling.com/blog/2025/i-have-nut-on-my-pi-so-my-servers-dont-die).

## License

GPLv3

## Author Information

This project was created in 2025 by [Jeff Geerling](https://www.jeffgeerling.com/).
