Gateway Patch Appliance
=======

This role will apply both platform and application patches for Layer7 appliances.  The sample playbook should execute in serial mode and reboot node after the installation of each patch.
 
 **Note: This role must be executed with root privileges.**

Requirements
------------
* Download patch files from [support site](https://support.ca.com/us/product-content/recommended-reading/technical-document-index/ca-api-gateway-solutions-and-patches.html)
* Add the patch files to the [roles/gateway_patch_appliance/files](roles/gateway_patch_appliance/files) directory
* The role must be executed with the Ansible user having root privilege.
* The Gateway nodes are added to [patch_hosts](inventories/demo/hosts) in HOSTS file

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


## gateway-ansible-playbook instructions

### Setup Controller:
    * Clone the [REPO](https://github.com/CAAPIM/gateway-ansible-playbook)
    * Follow instructions in [README.md](https://github.com/CAAPIM/gateway-ansible-playbook/#prerequisites)
    * Instructions on how to install sshpass:
```
    $ curl -O -L http://downloads.sourceforge.net/project/sshpass/sshpass/1.06/sshpass-1.06.tar.gz && tar xvzf sshpass-1.06.tar.gz
    $ ./configure
    $ make
    $ sudo make install
```

### Setup Project
*  Edit inventories/sample/hosts
```
    [gateway_primary_db]
    rhel.94.demo dest=centos.10.demo
    gateway_mysql_source]
    rhel.94.demo dest=centos.10.demo
    [gateway_mysql_dest]
    centos.10.demo source=rhel.94.demo
```

### Edit variables for the passwords and output directories in inventories/sample/group_vars/all.yml

     * Edit the hosts file at path inventories/sample/hosts file 
     * Add the vaulted passwords in the inventories/sample/group_vars/all.yml
         ` ansible-vault encrypt_string --vault-password-file vault-password-file.txt '7layer' `


### Enable OTK export by uncommenting the following lines in: inventories/sample/group_vars/gateway_mysql.yml
```
    otkdb_name: otk_db
    otkdb_user: otk_user
    otkdb_userpwd: '{{ vault_gateway_database_pass }}'
```

###  Check Inventory Hosts
    ` $ cd /Users/aric/Projects/gateway-ansible-playbook `
    ` $ ansible-inventory -i ./inventories/sample --list `


### Enable ssh access for root user:
```
    update /etc/ssh/sshd_config
        PermitRootLogin yes
    update /etc/ssh/ssh_allowed_users
        root added with ssgconfig
    service sshd restart
```


## Execute Playbooks - Source Gateways
```
$ ansible-playbook playbooks/gateway-preupgrade-analyzer.yml -i inventories/sample/hosts --vault-password-file vault-password-file.txt

$ ansible-playbook playbooks/gateway-database-export.yml -i inventories/sample/hosts --vault-password-file vault-password-file.txt

ROOT $ ansible-playbook playbooks/gateway-basic-backup.yml -i inventories/sample/hosts --vault-password-file vault-password-file.txt

ROOT $ ansible-playbook playbooks/gateway-restore-basic-backup.yml -i inventories/sample/hosts

```


## Execute Playbooks - New v10 CentOS

```
$ ansible-playbook playbooks/gateway-autoprovision-nodes.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

$ ansible-playbook playbooks/gateway-database-import.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

$ ansible-playbook playbooks/gateway-database-schema-upgrade.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

$ ansible-playbook playbooks/gateway-install-license.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

ROOT $ ansible-playbook playbooks/gateway-restore-basic-backup.yml -i inventories/demo/hosts

$ ansible-playbook playbooks/gateway-restart.yml -i inventories/demo/hosts

```

