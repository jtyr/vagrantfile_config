Vagrantfile Config
==================

This is a generic
[Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/) configurable
via config file. Current implementation supports [VirtualBox
provider](https://www.vagrantup.com/docs/virtualbox/) and [Ansible
provisioner](https://www.vagrantup.com/docs/provisioning/ansible.html).

Please report any issues or send PR.


Installation
------------

It's recomended to make a clone of this Git repository and symlink the
`Vagrantfile` into to the workdir:

```
# Clone the Git repo
git clone https://github.com/jtyr/vagrantfile_config.git
# Symlink the Vagrant file into the workdir
ln -s $PWD/vagrantfile_config/Vagrantfile /path/to/the/workdir/
# Change directory into the workdir
cd /path/to/the/workdir/
# Create minimal configuration for the Vagrantfile
cat > vagrant.yaml <<END
---

vms:
  test:
END
# Run Vagrant to spin up the VM
vagrant up
```

Like that, when the Git directory is updated, all symlinked Vagrantfiles
will be updated automatically as well.


Configuration
-------------

Configuration can be done on two levels - for all VMs in the `default`
section and for individual VMs in the `vms` section.

```
---

default:
  # Default box for all VMs is centos/7
  box: centos/7

vms:
  # VM test1 has no configuration. The default configuration will be used.
  test1:
  # VM test2 has custom configuration for the box name
  test2:
    box: centos/6
```

Example of more complex configuration:

```
---

default:
  # Box definition can be either string as shown above or hash allowing to
  # define additional parameters
  box:
    name: centos/7
    # Box version
    version: v1611.01
    # Custom URL where to download the box
    url: https://www.server.com/box/centos7.box
    # Disable SSL certificate checking if the cert is self-signed
    download_insecure: yes
  # Group all VMs into the following VirtualBox group
  group: TestGroup
  # Create secondary network interface with the following IP range
  ip_range: 192.168.10.%d
  # The IPs will start with 192.168.10.50
  ip_start: 50
  # Assign unique SSH port to each of the VMs starting at this number
  ssh_port_start: 10000
  # Optional SSH params
  ssh:
    # User used to connect via SSH
    user: vagrant
    # User password
    password: vagrant
    # Path to the private key
    private_key: ~/.vagrant.d/insecure_private_key
  # Enable to show the VM gui
  gui: yes
  # Use 2 CPU cores
  cpus: 2
  # All VMs will have 1GB of memory
  memory: 1024
  # Extra disks will use SATA controller
  storage_controller_type: sata
  # Name of the controller
  #storage_controller_type: SATA
  # Whether to create disk controller
  #storage_controller_create: yes
  # Disk device
  #storage_controller_device: 0
  # Disk port offset
  #storage_controller_offset: 0
  # All VMs will have two disks (system disk 20GB, data disk 10GB)
  extra_disks:
    - 20
    - 10
  # Enable shared folder for Linux guest
  synced_folder:
    enabled: yes
    host: /tmp
    guest: /vagrant
    #create: no
    #type: virtualbox
  # Enable shared folder for Windows guest
  #synced_folder:
  #  enabled: yes
  #  host: .
  #  guest: /vagrant
  #  # Options for the 'virtualbox' type
  #  type_opts:
  #    automount: yes
  # Expose some VM ports
  ports:
    # Expose port 80
    HTTP:
      host: 10080
      guest: 80
      # Optionally, also protocol can be specified [tcp|udp] - default is tcp
      proto: tcp
    # Expose port 443
    HTTPS:
      host: 10443
      guest: 443
  # Set MAC address of the second NIC
  #mac: 080027ad3020
  # Set Boot priority of the second NIC
  #bootprio: 1
  # Set environment variables
  #env_vars:
  #  ANSIBLE_PYTHON_INTERPRETER: /usr/local/bin/python
  # Enable or disable USB
  #usb: no
  # Specify USB version [1|2|3]
  #usb_version: 1
  # Audio setting
  #audio: none
  # Allow setting hostnames
  #set_hostname: no

vms:
  test1:
  # All default options from above can also be used on the VM level
  test2:
    # For example this VM will have 3 CPU cores
    cpus: 3
    # It's also possible to define explicit IP
    ip: 192.168.10.3
    # Explicit SSH port can be set per VM
    ssh:
      port: 1234
    # Set custom hostname
    #hostname: test2.local
```

Example of Ansible provisioning for all VMs:

```
---

default:
  # Provision all VMs with Ansible after all VMs are built
  provision_all: yes

vms:
  test1:
  test2:
```

Example of Ansible provisioning for individual VMs:

```
---

default:
  # Provision all VMs with Ansible after each VMs is built
  provision_individual: yes

vms:
  test1:
  test2:
```

More complex example of Ansible provisioning:

```
---

# Re-usable variable for default Ansible groups definition
default_groups: &default_groups

default:
  # Provision all VMs with Ansible after each VMs is built
  provision_individual: yes
  # Customize the Ansible provisioner
  privisioning:
    # All VMs will use test.yaml playbook
    playbook: test.yaml
    # Define some custom groups
    groups: *default_groups
      # Adding all hosts into the "local" group
      local:
        - test1
        - test2
      # Define default Python interpreter for all hosts
      local:vars:
        ansible_python_interpreter: /usr/bin/python2
    # Define default extra variables
    extra_vars:
      cloud_provider: local
      sudo_users__custom:
        - ansible:
            host: ALL
            runas: ALL
            tag: NOPASSWD
            cmd: ALL
    # Define some extra command line parameters for Ansible
    raw_arguments:
      - --diff
      - --forks=10

vms:
  test1:
    # This section can contain any option provided by the Ansible provisioner
    provisioning:
      # Put this host also into the "test" group
      groups:
        # First add the default groups defined above
        <<: *default_groups
        # Then add the "test" group and define the host list for it
        test:
          - test1
  test2:
```


License
-------

MIT


Author
------

Jiri Tyr
