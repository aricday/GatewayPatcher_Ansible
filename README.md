Gateway Patch Appliance
=======

This role will apply both platform and application patches for Layer7 appliances.  The sample playbook should execute in serial mode and reboot node after the installation of each patch.
 
 **Note: This role must be executed with root privileges.**

Requirements
------------
* Download patch files from [support site](https://support.ca.com/us/product-content/recommended-reading/technical-document-index/ca-api-gateway-solutions-and-patches.html)
* Add the patch files to the [roles/gateway_patch_appliance/files](roles/gateway_patch_appliance/files) directory
* The role must be executed with the Ansible user having root privilege.
* The Gateway nodes are added to [patch_hosts](inventories/demo/hosts#L28) in HOSTS file

```
    $ ansible-playbook playbooks/gateway-appliance-patcher.yml -i inventories/demo/hosts 
```

group_vars Variables
--------------
```
---
# local directory on controller
patch_source_directory: "<directory>/GatewayPatcher_Ansible/roles/gateway_patch_appliance/files/"

# application patch file name
application_patchId: 'CA_API_Gateway_v9.4.00.10242-CR04'

# plarform patch file name
platform_patchId: 'CA_API_PlatformUpdate_64bit_v9.X-RHEL-2020-02-25'

```

Example Playbook
----------------
```
---
- hosts: patch_hosts
  become: false
  serial: 1
  gather_facts: true
  roles:
    - gateway_patch_appliance

```

Video How-To
----------------
[![Watch the video](https://www.broadcom.com/media/1211233548536/esd-hero-1920x455_final.jpg)](https://broadcom.ent.box.com/file/643731820225)


