# Setup Ansible Deployer
Ansible playbooks to migrate Layer7 Gateways from RHEL to CentOS

## Setup Controller:
* Clone the [REPO](https://github.com/CAAPIM/gateway-ansible-playbook)

* Follow instructions in [README.md](https://github.com/CAAPIM/gateway-ansible-playbook/#prerequisites)

* Additional instructions on how to install sshpass:
```
	$ curl -O -L http://downloads.sourceforge.net/project/sshpass/sshpass/1.06/sshpass-1.06.tar.gz && tar xvzf sshpass-1.06.tar.gz
	$ ./configure
	$ make
	$ sudo make install
```

## Setup Project
### Edit the hosts file at path *inventories/sample/hosts*
```
    [gateway_primary_db]
    master.10.demo src=rhel.94.demo
    gateway_failover_db]
    slave.10.demo src=slave.94.demo
```

### Edit variables and output directories in *inventories/sample/group_vars/all/vars.yml*

### Add the vaulted passwords in *inventories/sample/group_vars/all/vault.yml*
```
	$ ansible-vault encrypt_string --vault-password-file vault-password-file.txt '7layer' 
```

### Enable OTK export by uncommenting the following lines in: *inventories/sample/group_vars/all/vars.yml*
```
	otkdb_name: otk_db
	otkdb_user: otk_user
	otkdb_userpwd: '{{ vault_gateway_database_pass }}'
```

###  Check Inventory Hosts
```
	 $ ansible-inventory -i ./inventories/sample --list
	 $ ansible-inventory -i ./inventories/sample --graph
```

## Execute Playbooks - Source Gateways

Analyze the gateways in scope for playbook.  Export report into [playbooks/report]() directory
```
$ ansible-playbook playbooks/gateway-preupgrade-analyzer.yml -i inventories/sample/hosts
```

Export the primary DB into [tmp/db_dump]() directory
```
$ ansible-playbook playbooks/gateway-database-export.yml -i inventories/sample/hosts
```

**ROOT USER** - ssgbackup files and assertions 
```
$ ansible-playbook playbooks/gateway-basic-backup.yml -i inventories/sample/hosts
```

**ROOT USER** - ssgrestore backup files and assertions (requires `upgrade_source` variable defined in HOSTS)
```
$ ansible-playbook playbooks/gateway-restore-basic-backup.yml -i inventories/sample/hosts
```

### If you need to enable ssh access for root user:
```
	update /etc/ssh/sshd_config
		PermitRootLogin yes
	update /etc/ssh/ssh_allowed_users
		root added with ssgconfig
	service sshd restart
```

## Execute Playbooks - New v10 CentOS

**ROOT USER**

Start replication on the 2 configured nodes as root
```
$ ansible-playbook playbooks/gateway-database-replication.yml -i inventories/demo/hosts
```
* After replication has started, use ssgconfig to point the slave node back to master and add failover in slave.
* Verify replication ` mysql -e "show slave status\G" `


Initial configuration of fresh install gateway
```
$ ansible-playbook playbooks/gateway-autoprovision-nodes.yml -i inventories/demo/hosts
```

Import the primary DB
```
$ ansible-playbook playbooks/gateway-database-import.yml -i inventories/demo/hosts
```

**ROOT USER** - ssgrestore backup files and assertions
```
$ ansible-playbook playbooks/gateway-restore-basic-backup.yml -i inventories/demo/hosts  --vault-password-file vault-password-file.txt
```

Upgrade DB Schema, Install License and Reboot nodes
```
$ ansible-playbook playbooks/gateway-database-schema-upgrade.yml -i inventories/demo/hosts

$ ansible-playbook playbooks/gateway-install-license.yml -i inventories/demo/hosts

$ ansible-playbook playbooks/gateway-restart.yml -i inventories/demo/hosts
```


## Maintain Patches

**ROOT USER** -  Run patcher for platform/appliance L7P files.
`  $ ansible-playbook playbooks/gateway-appliance-patcher.yml -i inventories/demo/hosts `
