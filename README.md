# SFS Infrastructure

Work-in-progress Ansible setup for [Seattle Flu Study
infrastructure](https://github.com/seattleflu/documentation/wiki/infrastructure).
Our intent is to describe our infrastructure setup with Ansible (and/or other
tools) over time, piece by piece.

Requirements for using this repository:

  - A local copy of the repository, e.g. checked out with git

  - Ansible 2.9+, installed with your preferred
    [installation method](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

  - Dependencies from Ansible Galaxy installed locally, using `ansible-galaxy
    install -r requirements.yaml`

  - For production deployments: credentials for our infrastructure, e.g. an
    authorized SSH keypair for backoffice.seattleflu.org

  - For local development: a [Multipass](https://multipass.run) instance named
    `backoffice` (see below)


## Organization

The layout of this repository is:

    ansible.cfg     — Global Ansible configuration options.

    *.yaml          — Playbooks (e.g. backoffice.yaml)

    inventory/       - An inventory of our hosts.
      …

    tasks/          — Conceptual groupings of tasks for use by playbooks
      …

    files/          — Static configuration files used by playbooks/tasks
      …

Each playbook is intended to (eventually) capture the whole setup of a host or
group of hosts.  If the playbook for a host becomes unwieldy, we can refactor
it into multiple files grouped by functionality.

Future additions should generally follow the [example
setup](https://docs.ansible.com/ansible/latest/user_guide/sample_setup.html) in
Ansible's documentation.


## Running playbooks

To test your connection to our hosts:

    ansible all -m ping -u ubuntu -i inventory

To run a playbook in development:

    ansible-playbook a-playbook.yaml -i inventory/development

To run a playbook in production, change the inventory used:

    ansible-playbook a-playbook.yaml -i inventory/production


## Writing playbooks

Each playbook is intended to (eventually) capture the whole setup of a host or
group of hosts.  To share tasks between playbooks, write a
[role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html).

Declarative configuration and idempotency are a key feature of Ansible (and
other similar tools).  When naming plays and tasks, keep this in mind by using
statements that declare the desired state ("service X is started") instead of
actions ("start service X").


## Secrets

Secrets such as passwords and API tokens are encrypted using [Ansible
Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) and
stored in this repo.  Ansible Vault uses strong symmetric encryption with a
single decryption password shared by all secrets stored in a logical "vault".
Ansible automatically decrypts the secrets as necessary when they're referenced
by playbooks but must be told how to obtain the vault password (e.g. by asking
you, by reading a file, or by running another program).  This vault password is
stored outside of Ansible, such as in your password manager of choice.

We currently use a single `seattleflu` vault, which should be used for all
secrets except in the very rare case of when the whole team should not have
access to a secret.  Use the `--vault-id seattleflu@prompt` option when running
`ansible`, `ansible-playbook`, or `ansible-vault` to both specify the
`seattleflu` vault and be asked for the vault password.

Please use encrypted files in preference to encrypted variables, as files are
easier to re-encrypt later with a new password (e.g. when someone departs the
team).

To encrypt a file:

    ansible-vault encrypt --vault-id seattleflu@prompt path/to/file

To check an encrypted file's contents:

    ansible-vault view --vault-id seattleflu@prompt path/to/file


## Multipass

[Multipass](https://multipass.run) is a really nice CLI and GUI tool to spin up
Ubuntu VMs on your computer no matter your host OS.  The idea is that for local
development and testing, this Ansible setup can target a local VM that closely
matches production.  For now "closely matches" means only the same Ubuntu
version, but this will naturally become higher fidelity as we describe more of
our infrastructure using Ansible!

Create a `backoffice` instance:

    multipass launch --name backoffice 18.04    # Ubuntu 18.04 LTS

Add your SSH public keys so that Ansible can login via SSH:

    ssh-add -L | multipass exec backoffice -- tee -a /home/ubuntu/.ssh/authorized_keys

Now test that Ansible can login by issuing an ad-hoc ping:

    ansible development -m ping -i inventory/development -u ubuntu

This specifies the *development* host group, which will be found in the default
inventory (*inventory*, which is a yaml file).  You should see green text saying something like:

    development | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        },
        "changed": false,
        "ping": "pong"
    }

For other troubleshooting, you can get a shell on your new instance at any point
with:

    multipass shell backoffice

If you want to start from a clean slate, delete the `backoffice` instance and
SSH's remembered host key (to avoid errors about the new instance's different
host key):

    multipass delete --purge backoffice
    ssh-keygen -R backoffice.multipass

Then re-create the instance as above.
