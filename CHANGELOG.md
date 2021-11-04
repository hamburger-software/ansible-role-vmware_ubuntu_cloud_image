# Changelog

## v5.2.0 (2021-11-04)

### Features

- Virtual Machines can be placed in folders.

## v5.1.1 (2020-12-14)

### Features

- Ubuntu 20.10 is a tested platform.
- Fixed an inaccurate task name.

## v5.1.0 (2020-08-24)

### Features

- VM notes can be set with `annotation`.

## v5.0.0 (2020-08-24)

### Changes

- Because of renamed dependencies, the module now requires Ansible 2.9.0 or above.

### Bugfixes

- Remove warning about changed default permissions on file creation.

### Features

- [customvalues](https://stackoverflow.com/a/57976458/2402612) can be set on VM creation.

This can be used to avoid [excessive syslog messages from multipathd](https://bugs.launchpad.net/ubuntu/+source/multipath-tools/+bug/1875594) on Ubuntu 20.04 based VMs.
Add this fragment to the role configuration: 

```yaml
customvalues:
  - key: disk.EnableUUID
    value: 'TRUE'
```

## v4.0.1 (2020-05-13)

### Bugfixes

- The domain search no longer is ignored.

### Features

- Ubuntu 20.04 is a tested platform.

## v4.0.0 (2019-11-06)

### Changes

- **The role was relocated to the new Galaxy namespace [hamburger_software](https://galaxy.ansible.com/hamburger_software).**

## v3.0.0 (2019-05-17)

### Features

- Disks may be added.  
  In order to add a disk, supply a `disk` parameter with two disk specifications.
  The first element pertains to the configuration of the existing disk.
  The second and subsequent ones configure additional disks.
  See [vmware_guest_disk](https://docs.ansible.com/ansible/latest/modules/vmware_guest_disk_module.html).
- Ubuntu 19.04 is a tested platform.

### Changes

- In order to enable creation of disks, a new module (which was added in Ansible 2.8.0) had to be used.
  This module requires additional parameters when used to configure an existing disk.
  If your playbooks increase disk size, they will have to be updated.
  See the [example playbook](README.md#example-playbook).

## v2.1.0 (2019-04-02)

### Features

- Disk size may be increased.
- Support for custom network mappings
  
  This allows to override the default network mappings of
  *ansible_deploy_ovf* with user-defined or empty mappings.

## v2.0.1 (2018-12-17)

### Features

- Parameter verification is only executed once.

### Changes

- Ansible 2.7.1 is required, a workaround for Ansible 2.7.0 was removed.

## v2.0.0 (2018-10-02)

### Features

- Customization of the ssh public keys was moved from the dedicated `public-keys` property to a [Cloud Config Data](https://cloudinit.readthedocs.io/en/latest/topics/format.html#cloud-config-data) section file specified in `user-data`.
This enables the following features: 
  - Multiple ssh public keys can be added. 
  - A password can be set for the user "ubuntu" without forcing it to be changed on first login (which would break the ansible connection).

- A workaround for https://github.com/ansible/ansible/issues/45223 was added.

### Changes

- The `cidr` variable was renamed to `network`.

## v1.0.0 (2018-09-27)

Support for creating virtual machines from Ubuntu Cloud Images with adjusted hardware.
