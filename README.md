# Essentials

Ansible role that configures essential system components, such as:

- creating custom files and directories (e.g. drop-in `foo.conf.d` files)
- systemd user-scoped services prep
- system controls (`sysctl`)

## Requirements

There are no requirements for this, it should work out of the box with Ansible.

## Role Variables

Please see `defaults/main.yml` for variables that can all be set on a per-host
basis.

### Files and Directories

Here's an example of how to use the `files` and `directories` tasks. These are
very simple and allow Ansible to control drop-in files and pretty much anything
else.

Define these variables in your host(s):

```yaml
directories:
  - dir: /etc/systemd/system/sshd.service.d
    become: true
    owner: root
    group: root
    mode: "0644"
    # no_log: true  # for sensitive directories, you can hide log output

files:
  - dest: /etc/systemd/system/sshd.service.d/override.conf
    become: true
    owner: root
    group: root
    mode: "0644"
    # no_log: true  # for sensitive files, you can hide log output
    content: |
      [Unit]
      Wants=network-online.target
      After=network-online.target
```

You can also leave them undefined or as an empty array.

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
ansible-playbook site.yml --tags base,sysctl,files,directories --step
```

## License

MIT

## Author Information

<https://charlesmknox.com>
