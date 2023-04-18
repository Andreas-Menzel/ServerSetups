# ServerSetups - Ansible

## Roles

Here is a list of the roles from `roles/` with information on what they do and
how to call them.

### setup_users

#### Note

- When calling this role, make sure to set `gather_facts: false` or else
  it will throw an error for the initial setup.
- This role is only compatible with Debian and Fedora systems. If you want to
  use something else, you have to slightly modify the
  `SET sudoers_group BASED ON DISTRIBUTION` section in the roles file.

#### Description

This role will try to connect to the server with the automation user and
ssh-key. If this fails (for example if the server is brand new and does not
have the automation user yet), it will connect with the initial user and the
initial password. (At the end of this role, the user is switched back to the
automation user)

It will create an admin user and an automation user (without a password). Both
users will be added to the sudoers group (*sudo* on Debian, *wheel* on Fedora).

The ssh public keys of the admin user will be added to the authorized_keys of
the admin user **and** the initial user.

The ssh public keys for the automation user will be added to the authorized_keys
of the automation user.

The specified main users will be created and the ssh public key will be added to
the authorized_keys of the respective user. Each user will be added to the
specified groups and given root permissions if the `is_sudoer` variable is set
to true.

The role will enable passwordless sudo for the sudoers group and disable root
login and login with password via ssh.

#### Configuration

The configuration is done in `inventory/hosts.yml`.

The initial user and password are only needed for the initial setup. After that
you can change the password. Please note that ssh authentication with password
will be disabled, so it *should* be ok to not change it.

**Note: The initial user must be able to execute a command with sudo without**
**the need for a password.**

```
initial_user: "root"
initial_pass: "InitialSecretP@ssword!"
```

The admin user is intended for normal system administrator access or as a backup
user with root access.

```yml
admin_user:
  name: server
  ssh_keys:
    files:
      - "~/.ssh/id_rsa.pub"
      - "/path/to/other/public/key"
    strings:
      - "ssh-ed25519 KEY_1"
      - "ssh-ed25519 KEY_2"
      - "ssh-ed25519 KEY_3"
```

The automation user is used by Ansible (and is intended to be also used by
other automation scripts). Make sure to add the key of the machine executing
(this) Ansible playbook.

```yml
automation_user:
  name: automation
  ssh_keys:
    files:
      - "~/.ssh/id_rsa.pub"
      - "/path/to/other/public/key"
    strings:
      - "ssh-ed25519 KEY_1"
      - "ssh-ed25519 KEY_2"
      - "ssh-ed25519 KEY_3"
```

Now you can specify the main ("normal") users. If a user should have root
permissions, ONLY set the `is_sudoer` variable to true (don't add the sudoers
group manually).

```yml
users:
  - name: user1
    public_key: "ssh-ed25519 KEY"
    is_sudoer: true
    groups:
      - "optional_group_1"
      - "optional_group_2"
  - name: user2
    public_key: "ssh-ed25519 KEY"
    is_sudoer: false
    groups:
```

### setup_packages_apt / setup_packages_dnf

#### Description

Installs (and updates) packages.

#### Configuration

You can select, if the packages should be updated:

```yml
update_packages: true
```

Select the packages to install:

```yml
packages:
  - nano
  - micro
  - pip
  - git
  - curl
```

### install_golan

#### Note

GO will **not** be (re)installed, if it is already installed (`/usr/local/go`
already exists). If you want to update GO, you have to manually uninstall GO,
update the `go_version` variable and then execute this role.

#### Description

Downloads and installs GOlan.

#### Configuration

You can specify the version of GO:

```yml
go_version: 1.20.3
```
