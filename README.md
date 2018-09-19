# CentOS 7 Common Initial Setup Tasks

This role is for hardening default SSH security in CentOS 7.

## Requirements

`mariuszczyz.centos-common` role is required if `DISABLE_PASSWORD` will be enabled.

## Role Variables

`DISABLE_ROOT` - set to true if you want to disable root SSH login
`ADJUST_IDLE`  - set to true if you want to enable idle session timeout
`ALIVE_INTERVAL` - set to a numerical value in seconds, 300 is a good and balanced target
`ALIVE_COUNT_MAX` - set to a numerical value in seconds
`LIMIT_USERS` - add all usernames allowed to SSH into the server
`CHANGE_SSH_PORT` - security by obscurity but anything is better than nothing in limiting SSH service from port scans
`DISABLE_PASSWORD` - setting this will make sure only the SSH key owner will be able to SSH, the same way AWS does it

## Dependencies

`mariuszczyz.centos-common` role is required if `DISABLE_PASSWORD` will be enabled. Once password autehntication is disabled only SSH key based authentication will be available and that requires the public SSH key is present on the server. That step is taken care of in the `mariuszczyz.centos-common` role.

## Example Playbook

Fetch this role from Ansible Galaxy:

`ansible-galaxy install mariuszczyz.centos-sshd-hardening`

In playbook.yml:

```bash
- hosts: servers
  roles:
    - { role: mariuszczyz.centos-sshd-hardening, tags: ['centos-sshd-hardening'] }
```

Before any SSH hardening configuration changes are applied this ansible-playbook command should be ran (as root):

`ansible-playbook -i hosts playbook.yml --user root --ask-pass --limit=hostname`

After SSH hardening configuration changes are made which prohibit root login run ansible-playbook as regular user:

`ansible-playbook -i hosts playbook.yml --user username --become --ask-sudo-pass --limit=hostname`

## License

BSD

## Author Information

Author: Mariusz Czyz
Date: 09/2018
