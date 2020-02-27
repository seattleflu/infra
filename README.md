# Ansible

## Installation

Followed [these instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu).

## Create a basic inventory

In `/etc/ansible/hosts`:

```ini
[dev]
localhost

[prod]
backoffice.seattleflu.org
```

## Create a basic playbook

See `example.yml`.
