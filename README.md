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

Here's an example of how to use the `files`, and `systemd-services` tasks. These
are very simple and allow Ansible to control drop-in files and pretty much
anything else.

Define these variables in your host(s):

```yaml
files:
  - dest: /etc/systemd/system/sshd.service.d
    state: directory
    become: true
    owner: root
    group: root
    mode: "0644"
    # no_log: true  # for sensitive directories, you can hide log output
    # If desired, you can execute an arbitrary command that will execute after
    # all directories finish getting created. Can optionally specify executable,
    # chdir, and become for this as well - these get passed directly to
    # ansible.builtin.shell.
    # run_command_after:
    #   shell: |
    #     podman unshare chown -R nobody:nobody /path/to/directory

  - dest: /etc/systemd/system/sshd.service.d/override.conf
    become: true
    # All supported options are also passed to an ansible.builtin.file task if
    # state is defined this lets you get creative and even allows directories to
    # be created, if you prefer this instead of using the separate directories
    # tasks above. See files.yml for how options are mapped, it's pretty simple.
    #
    # The only caveat is that values will not get processed in the order you
    # expect. All of the items in this "files" list with a defined "state" value
    # will get processed first, and then the other values will be processed
    # after.
    state: absent

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

systemd_services:
  - name: scheduled-backups.timer
    scope: user
    state: started
    enabled: true
    daemon_reload: true
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
ansible-playbook site.yml --tags base,sysctl,files,systemd-services --step
```

## License

MIT

## Author Information

<https://charlesmknox.com>
