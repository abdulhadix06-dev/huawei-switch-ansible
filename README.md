# Huawei Switch Ansible Playbooks

Ansible playbooks for common day-to-day tasks on Huawei VRP switches
(for example, the S5735 series), using the `community.network` collection.
They are written for use from the command line or from Semaphore
(a free, self-hosted Ansible UI), as a free alternative to a paid
controller for small teams.

## What is included

- `port-admin.yml` - shut down or bring up a switch port, then save only if changed
- `vlan-create.yml` - create a VLAN, give it a description, and optionally assign an access port to it
- `user-create.yml` - create a local SSH user in AAA view with a privilege level
- `config-push.yml` - push the same config lines to all switches in the inventory, then save only where changed
- `inventory.ini.example` - example inventory to copy and adapt
- `requirements.yml` - Ansible collection requirement

## Requirements

- Ansible
- `community.network` collection (see `requirements.yml`)
- SSH access to the switches. In the example inventory the SSH port is
  2022. Change `ansible_port` in your own inventory if your switches
  use a different port.

Install the collection with:

```
ansible-galaxy collection install -r requirements.yml
```

## Setup

1. Copy the example inventory and edit it with your own switches:

```
cp inventory.ini.example inventory.ini
```

2. Replace the example hostnames and example IPs (`192.0.2.x`, which are
   documentation-only addresses) with your real switch addresses.
3. Do NOT put a real password in the inventory file, and do NOT commit
   your real `inventory.ini` to Git. It is already listed in
   `.gitignore`. Use a prompt, an SSH key, or Ansible Vault for
   credentials instead.

## Usage

Run a playbook against your inventory, passing the values as extra
variables. Examples:

Port shut / unshut:

```
ansible-playbook -i inventory.ini port-admin.yml -e '{"target_port": "GigabitEthernet0/0/5", "port_action": "shutdown"}'
```

Create a VLAN:

```
ansible-playbook -i inventory.ini vlan-create.yml -e '{"vlan_id": 150, "vlan_name": "Printers"}'
```

Create a VLAN and assign an access port to it:

```
ansible-playbook -i inventory.ini vlan-create.yml -e '{"vlan_id": 150, "vlan_name": "Printers", "access_port": "GigabitEthernet0/0/12"}'
```

Create a user (pass the password at run time, do not write it in a file):

```
ansible-playbook -i inventory.ini user-create.yml -e '{"new_username": "newuser", "new_password": "use-a-prompt-or-vault-value-here", "priv_level": 15}'
```

Push config lines to all switches:

```
ansible-playbook -i inventory.ini config-push.yml -e '{"config_lines": ["description Test"], "save_config": true}'
```

To skip saving, pass `"save_config": false`.

In Semaphore, create a task template for a playbook, select your
inventory, and put the same JSON in "Extra variables".

## Safety notes - please read before first use

- These playbooks were written for Huawei VRP switches. They have not
  been tested on every model or firmware version. Test first on ONE
  switch, on an empty or test port, with a small change.
- For VLAN names, do not use spaces. Use an underscore, for example
  `Sales_Team`.
- The user-create playbook passes the new password as an extra
  variable, so it can appear in Semaphore task history. Delete that
  task run after the work is done.
- A switch stores a user password as a hash, so re-running
  user-create will usually report CHANGED even if the user already
  exists. That is normal for VRP.
- Nothing changes on a switch until you actually run a playbook.
  Copying files or creating a Semaphore template alone changes nothing.

## License

MIT. See `LICENSE`.
