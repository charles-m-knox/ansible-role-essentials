# Essentials

Ansible role that configures essential system components, such as:

- systemd user-scoped services prep
- system controls (`sysctl`)

## Requirements

There are no requirements for this, it should work out of the box with Ansible.

## Role Variables

Please see `defaults/main.yml` for variables that can all be set on a per-host
basis.

## Dependencies

None at the moment.

## Usage

First, create a `requirements.yml` in your Ansible setup if you haven't already,
here's an example:

```yaml
---
collections:
  - name: community.general
  - name: ansible.posix
  - name: community.crypto

roles:
  - name: charles-m-knox.essentials
    src: https://github.com/charles-m-knox/ansible-role-essentials.git
    scm: git
    version: main
```

Next, install it:

```bash
# include the -f flag to forceably re-install and get the latest (equivalent to
# upgrading)
ansible-galaxy role install -f -r requirements.yml
```

Now, in your `site.yml` (or whatever your playbook is named), use the role -
note that root access is required for some of the steps:

```yaml
- name: essentials
  hosts: all
  roles:
    - charles-m-knox.essentials
```

```bash
ansible-playbook site.yml --tags base,sysctl --step

# you can also just use the essentials tag:
ansible-playbook site.yml --tags essentials --step
```

## License

MIT

## Author Information

<https://charlesmknox.com>
