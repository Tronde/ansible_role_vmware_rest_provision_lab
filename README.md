vmware_rest_provision_lab
=========================

This role helps you to provision a lab environment consisting of one or more virtual machines to a VMware vSphere Cluster (vSphere 7.0.2 or greater).

I wrote this role with the following use case in mind.

For different scenarios I need lab environments based on Fedora, CentOS or RHEL operating systems (OS). These scenarios are sometimes recurring, so I'd like to deploy the same lab infrustructe again at a future point in time, with the same count of VMs but with a fresh OS installation.

I use variable files to specify the virtual machines (VM) for my labs and include them in this role as needed to provision a defined lab environment. This role focuses on creating the VMs.

For the guest OS installation I use ISO images with a kickstart file included. When a provisioned VM is powered on with the ISO image mounted, the kickstart installation starts the unattended installation process. At the end of the process I have network connected VMs. From here on I could run additional roles or playbooks to configure the guest OS the way I need them for different scenarios.

*Disclamer:* The intended of this role is to create lab environments **not** production stable ones. You use this role at your owen risk. Please keep that in mind.

Requirements and Dependencies
-----------------------------

This role requires/depends on:

 - The `vmware.vmware_rest` collection
 - Python 3.6 or greater
 - `aiohttp`

Install the dependencies
^^^^^^^^^^^^^^^^^^^^^^^^

You can either install `aiohttp` using your OS package manager or using Python virtual environment.

Notes: For RHEL, there is no `python3-aiohttp` package available (yet), you can either get it from EPEL or install `aiohttp` using pip.

Installing the Collection from Ansible Galaxy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before using this role, you need to install the required collection with the `ansible-galaxy` CLI:

~~~
ansible-galaxy collection install vmware.vmware_rest
~~~

You can also include it in a `requirements.yml` file and install it via `ansible-galaxy collection install -r requirements.yml` using the format:

~~~
collections:
- name: vmware.vmware_rest
~~~

Role Variables
--------------

All input variables are specified in `defaults/main.yml`. You have to define these variables with values that are valid for your environment. Following you will find all the varialbes from `defaults/main.yml` with the short explanation.


Required information about the vSphere environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`vmware_rest_provision_lab_vc:` FQDN of the vCenter instance.
`vmware_rest_provision_lab_dc:` Name of the virtual Datacenter.
`vmware_rest_provision_lab_cluster:` Name of the vSphere Cluster.
`vmware_rest_provision_lab_user:` Username to connect to REST API; It's recommended to put this into a vault file.
`vmware_rest_provision_lab_pass:` Password to authenticate at REST API, It's recommended to put this into a vault file.
`vmware_rest_provision_lab_validate_certs: true` Boolean (true/false); whether to validate TLS certs or not; defaults to `true`.
`vmware_rest_provision_lab_datastore:` The name of datastore to create the VM in.
`vmware_rest_provision_lab_network:` The network name the VM should connect to.

ISO image to use for guest OS installation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`vmware_rest_provision_lab_iso_path:` Specifies the path to the ISO image to use.
Take the path of the ISO image from vSphere Client. It's usually something like '[DatastoreName] <UUID>/image.iso' or '[DatastoreName] MyFolder/image.iso'.

VMs are defined as dicts
^^^^^^^^^^^^^^^^^^^^^^^^

These should be defined in variable files, e.g. `vars/rhel_lab.yml` or in your playbook.

Example dictionary:

~~~
vmware_rest_provision_lab_guests:
  VM_NAME_1:
    guest_OS: RHEL_8_64 # See vmware.vmware_rest.vcenter_vm for possible values.
    hardware_version: VMX_19 # See https://kb.vmware.com/s/article/1003746
    hot_add_enabled: true
    size_MiB: 2048
    disk_size: 100000000 # Size in Bytes
    storage_policy_spec: thin 
  VM_NAME_2:
    guest_OS: RHEL_8_64 # See vmware.vmware_rest.vcenter_vm for possible values.
    hardware_version: VMX_19 # See https://kb.vmware.com/s/article/1003746
    hot_add_enabled: true
    size_MiB: 2048
    disk_size: 100000000 # Size in Bytes
    storage_policy_spec: thin
~~~

Example Playbook
----------------

The most simple version is to run:

~~~
- hosts: localhost
  roles:
     - vmware_rest_provision_lab
~~~

A playbook containing necessary vars could look like:

~~~
---
- hosts: localhost
  vars_files:
    - my.vault
  vars:
    vmware_rest_provision_lab_iso_path: >
      [vsanDatastore] MyFolder/rhel-8.7-x86_64-kickstart-dvd.iso
    vmware_rest_provision_lab_guests:
      test_vm1:
        guest_OS: RHEL_8_64
        hardware_version: VMX_19 # See https://kb.vmware.com/s/article/1003746
        hot_add_enabled: true
        size_MiB: 2048
        disk_size: 100000 # Size in Bytes
        storage_policy_spec: thin
      test_vm2:
        guest_OS: RHEL_8_64
        hardware_version: VMX_19 # See https://kb.vmware.com/s/article/1003746
        hot_add_enabled: true
        size_MiB: 2048
        disk_size: 100000 # Size in Bytes
        storage_policy_spec: thin
  roles:
    - vmware_rest_provision_lab
~~~

This playbook would provision two VMs `test_vm1` and `test_vm2`. Some of the variables needed are defined in the encrypted file `my.vault`.

Contribution and Feedback
-------------------------

Open Source lives on participation. In case you find any bug, error or other issue while using this role, please raise an issue and address it. You have a fix for it too? Then I'm looking forward to receiving your pull request.

In case you just wanna ask a question, please raise in issue for it as well and we could discuss it there.

License
-------

BSD

Author Information
------------------

Author: Joerg Kastning
