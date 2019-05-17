vmware_ubuntu_cloud_image
=========================

Ansible role for creating virtual machines based on [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/) in a vSphere environment.

### Ubuntu Cloud Images

Ubuntu offers pre-installed images for usage in clouds. 
One of the available image formats is _Open Virtualization Appliance_ (OVA) that can be imported into VMware.
The images use the [cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) mechanism to allow very basic configuration.
Sadly, there is no support for using static IP addresses and for adjusting the hardware during machine creation.

This role adds support for these features.

### Features

- Creates a virtual machine (VM) from a previously downloaded OVA file.
- Sets the hostname.
- Adds one or more ssh public keys and/or a password for the default user "ubuntu" so that Ansible can connect to the new machine.
- Optionally adjusts the hardware, e.g. number of CPUs or memory, see [vmware_guest](https://docs.ansible.com/ansible/latest/modules/vmware_guest_module.html#parameters) for possible customizations.
- Disk size may be increased (defaults to 10GB)
- Optionally changes the dynamic IP address to a static one (taken either from the playbook or from DNS).
- The VM is turned on and can be used in the same playbook that invoked this role.
- Several VMs can be created in parallel.
- Tested with Ubuntu Cloud Images [19.04](https://cloud-images.ubuntu.com/releases/19.04/release-20190514/ubuntu-19.04-server-cloudimg-amd64.ova), [18.10](https://cloud-images.ubuntu.com/releases/18.10/release-20190514/ubuntu-18.10-server-cloudimg-amd64.ova), [18.04](https://cloud-images.ubuntu.com/releases/18.04/release-20190514/ubuntu-18.04-server-cloudimg-amd64.ova) and [17.10](http://cloud-images-archive.ubuntu.com/releases/artful/release-20180706/ubuntu-17.10-server-cloudimg-amd64.ova).
  Older versions do not work because they do not use `netplan` for network configuration.

Requirements
------------

To use this role,

- You need a VMware vSphere environment where the VM will be deployed.
- Credentials for the vCenter server of that environment with appropriate permissions, see below.
- Download an OVA file, e.g. [ubuntu-18.04-server-cloudimg-amd64.ova](https://cloud-images.ubuntu.com/releases/18.04/release/ubuntu-18.04-server-cloudimg-amd64.ova) to the control machine.

If you want to retrieve the VM's IP addresses from DNS, you also have to 

- Install dnspython (python library, http://www.dnspython.org/) on the control machine.
- Use fully qualified domain names (FQDN, e.g. host.example.org) in the inventory.
  The FQDN will also be used as the VM name.
- Add A records for each VM you want to create.

The minimum Ansible version is 2.8.0.

### vSphere Permissions

The minimum permissions to create a VM with this role are:

    DataStore > Allocate Space
    Network > Assign Network
    Resource > Assign Virtual Machine to Resource Pool
    vApp > Import
    Virtual Machine > Interaction > Power On
    Virtual Machine > Configuration > Add New Disk

To adjust CPU and memory settings, you need

    Virtual Machine > Configuration > Change CPU count
    Virtual Machine > Configuration > Memory

To adjust disk size, you need

    Virtual Machine > Configuration > Extend virtual disk


Role Variables
--------------

### vCenter Connection

- The URL of the vCenter server is set with `vcenter_hostname` or the environment variable `VMWARE_HOST`.
- The vCenter user is set with `vcenter_username` or the environment variable `VMWARE_USER`.
- The vCenter password is set with `vcenter_password` or the environment variable `VMWARE_PASSWORD`.
- Certificate validation can be disabled by setting `vcenter_validate_certs=no` or setting the environment variable
 `VMWARE_VALIDATE_CERTS` to `no`.

### VMware Settings

- The OVA file on the control machine is specified with `ova_file`.
- The VM is created in the datacenter `vmware_datacenter` on the datastore `vmware_datastore`.
- The VM can be placed in a resource pool by specifying `vmware_resource_pool`.
- The VM name is `inventory_hostname` by default. It can be changed with `vm_guestname`.

### VM Settings

- The machine's hostname is `inventory_hostname_short` by default. It can be changed with `vm_hostname`.
- Use `ssh_keys` to set a list of public keys that will be added to the *authorized_keys* file of the user "ubuntu".
  At least one of `ssh_keys` and `password` has to be specified so that Ansible can connect to the new machine.
- Use `password` to set a password for the user "ubuntu".
  At least one of `ssh_keys` and `password` has to be specified so that Ansible can connect to the new machine.
- The hardware can be specified with `hardware`, containing a dictionary as specified in [vmware_guest](https://docs.ansible.com/ansible/latest/modules/vmware_guest_module.html#parameters).
- Disk size may be adjusted with `disk`. This parameter accepts a list of disk specifications as documented in [vmware_guest_disk](https://docs.ansible.com/ansible/latest/modules/vmware_guest_disk_module.html#parameters).
  The first disk corresponds to the imported virtual disk. Its size may only be increased.
  See the example playbook below for usage.
- User defined network mappings can be specified with `networks`, see [vmware_deploy_ovf](https://docs.ansible.com/ansible/latest/modules/vmware_deploy_ovf_module.html#parameters) for semantics.

To use a static IP address, use the following keys in the dictionary `static_ip`:
- `ipv4` - a specific IPv4 address you want to assign. Defaults to the IPv4 address found in DNS for the FQDN.
- `netmask` - the netmask in [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) notation, defaults to `8`.
- `gateway` - the default gateway (required)
- `dns_servers` - a list of the DNS servers' IP addresses, defaults to Google's public DNS servers.
- `dns_search` - a list of domain names that should be used as DNS search suffixes.
   Use this to put your VM in a domain. 

### Inventory Settings

As the VMs do not exist yet, the ssh server's key is unknown.
In order to connect to the new VMs, you need to turn off ssh host key checking.
If you plan to repeatedly create VMs with the same FQDNs, ssh should not store the fingerprints in the known_hosts file.

Therefore the recommended host/group variables are:

    ansible_user=ubuntu
    ansible_ssh_extra_args=-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null

Dependencies
------------

None.

Example Playbook
----------------

playbook:

    - name: Deploy a Ubuntu Cloud Image Virtual Appliance
      hosts: cloudimg
      gather_facts: no
    
      roles:
        - role: albers.vmware_ubuntu_cloud_image
          vars:
            vcenter_hostname: vcenter.your.domain
            vcenter_username: Administrator@vsphere.local
            vcenter_password: verysecret
            vcenter_validate_certs: no
            vmware_datacenter: your-datacenter
            vmware_datastore: your-datastore
            ova_file: ubuntu-18.04-server-cloudimg-amd64.ova
            hardware:
              num_cpus: 4
              memory_mb: 2048
            disk:
              - size_gb: 20
                datastore: your-datastore
                scsi_controller: 0
                unit_number: 0
              - size_mb: 250
                datastore: your-datastore
                scsi_controller: 0
                unit_number: 1
                type: thin
            static_ip:
              netmask: 16
              gateway: 10.0.42.1
              dns_servers: [10.0.47.11, 10.0.48.12]
              dns_search:
              - your.domain
            ssh_keys:
              - ssh-rsa AAAAB3Nz[...]== some-key-name
            password: passw0rd

inventory with 5 hosts:

    [cloudimg]
    vm-[1:5].your.domain
    
    [cloudimg:vars]
    ansible_user=ubuntu
    ansible_password=passw0rd
    ansible_ssh_extra_args=-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null

License
-------

MIT

Author Information
------------------
This role was created by Harald Albers at HS - Hamburger Software GmbH & Co. KG.
