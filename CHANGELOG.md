# Changelog

## v6.1.0 (2025-08-23)

Added support for more recent Ubuntu versions (22.04, 24.04, 25.04), dropped support for outdated Ubuntu versions.

## v6.0.1 (2025-03-09)

### Bugfixes

The role no longer uses the `include` task because it was removed from Ansible core.

## v6.0.0 (2021-12-22)

### Changes

[community.vmware/#179](https://github.com/ansible-collections/community.vmware/pull/179) changed the semantics of *vmware_guest*'s `customvalues` parameter and added `advanced_settings`.
As *hamburger_software.vmware_ubuntu_cloud_image* exposes `customvalues`, some usages will have to be adjusted.

For example, if you used
```yaml
  customvalues:
    - key: disk.EnableUUID
      value: 'TRUE'
```
to set configuration file parameters, change it to the new parameter
```yaml
  advanced_settings:
    - key: disk.EnableUUID
      value: 'TRUE'
```
`customvalues` is still present, see the vSphere documentation of [custom attributes](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vcenterhost.doc/GUID-73606C4C-763C-4E27-A1DA-032E4C46219D.html) and VM [configuration file parameters](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.hostclient.doc/GUID-8C639077-FF16-4D5D-9A7A-E16902CE00C2.html) for the difference between both parameters.

`advanced_settings` was added to *vmware_guest* in the [1.8.0 release](https://github.com/ansible-collections/community.vmware/blob/main/CHANGELOG.rst#v1-8-0) of the _community.vmware collection_.
This release is part of the _Ansible community package_ 3.2.0, which is based on _ansible-core_ 2.10.7 (see [build data](https://github.com/ansible-community/ansible-build-data/blob/main/3/CHANGELOG-v3.rst#ansible-base-4)).

For that reason, the minimum Ansible version of this role was raised to 2.10.7.

### Contributors

@dotalchemy ([#5](https://github.com/hamburger-software/ansible-role-vmware_ubuntu_cloud_image/pull/5))

## v5.2.0 (2021-11-04)

### Features

- Virtual Machines can be placed in folders.
- Ubuntu 21.04 is a tested platform.

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
