This ansible project is a public version of configuration scripts for home server OpenBSD routers, configuring pf and unbound.

I used a [PCEngines Router](https://pcengines.ch/), and then a [Protectli Router](https://kb.protectli.com/kb/how-to-install-openbsd-on-the-vault-2/) later.

## Preparation

### OpenBSD

Before running Ansible, you have to install python on the target node. You can use ansible to do so or just ssh directly and run the `pkg_add -I python%3` command.

```
ansible -m raw -a "pkg_add -I python%3" server1.local
```

Python will be at `/usr/local/bin/python3` .

Always use `--become-method=doas`

0-hosts.ini has a group with variables configuring openbsd.

```
ansible-playbook playbooks/daily.yml -i 'server1.local,' -i 'inventories/0-hosts.ini' -K
```

https://felix-kling.de/blog/2018/ansible-openbsd.html

https://echothrust.github.io/howtos/openbsd/Manage%20OpenBSD%20hosts%20using%20ansible.html

## Usage

### OpenBSD Daily

1. You can use -i 'localhost' to administer locally.

2. Apply at least Ansible users playbook to configure users with ssh keys, home directories, and uid/gid values, particularly the admins.

```
ansible-playbook playbooks/users.yml -i 'localhost,' -K
```

3. Using the root account, use `sudo passwd YOURUSERNAME` for any users with sudo permissions. No password hashes are stored in the playbook.

4. Apply the daily base playbook. This is intended in servers to run daily to enforce configurations and package lists are installed, but in desktops only needs to be run whenever you want.

```
ansible-playbook playbooks/daily.yml -i 'localhost,' -K -e "ansible_ssh_user=ansible ansible_ssh_private_key_file=~/.ssh/id_rsa"
```

You will want to schedule this playbook with Gitlab CI to run daily. If you do, you will probably want to use the rootless sudo ansible user with an ssh key on it.

```
-e "ansible_ssh_user=ansible ansible_ssh_private_key_file=~/.ssh/id_rsa"
```

### OpenBSD Hourly

You can also schedule an hourly equivalent specifically designed to keep systemd services up, just call it hourly.yml. Make sure that all jobs take less than an hour to run.
