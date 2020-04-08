# Setup Ansible Deployer
Ansible playbooks to migrate Layer7 Gateways from RHEL to CentOS

## Setup Controller:
* Clone the [REPO](https://github.com/CAAPIM/gateway-ansible-playbook)

* Follow instructions in [README.md](https://github.com/CAAPIM/gateway-ansible-playbook/#prerequisites)

* There are instructions on how to install sshpass here:
```
	$ curl -O -L http://downloads.sourceforge.net/project/sshpass/sshpass/1.06/sshpass-1.06.tar.gz && tar xvzf sshpass-1.06.tar.gz
	$ ./configure
	$ make
	$ sudo make install
```

## Setup Project
### Edit inventories/sample/hosts
```
	[gateway_primary_db]
	rhel.94.demo dest=centos.10.demo
	gateway_mysql_source]
	rhel.94.demo dest=centos.10.demo
	[gateway_mysql_dest]
	centos.10.demo source=rhel.94.demo
```

### Edit variables for the passwords and output directories in *inventories/sample/group_vars/all.yml*

	 * Edit the hosts file at path inventories/sample/hosts file 
	 * Add the vaulted passwords in the inventories/sample/group_vars/all.yml
	 	` ansible-vault encrypt_string --vault-password-file vault-password-file.txt '7layer' `


### Enable OTK export by uncommenting the following lines in: *inventories/sample/group_vars/gateway_mysql.yml*
```
	otkdb_name: otk_db
	otkdb_user: otk_user
	otkdb_userpwd: '{{ vault_gateway_database_pass }}'
```

###  Check Inventory Hosts
	` $ cd /Users/aric/Projects/gateway-ansible-playbook `
	` $ ansible-inventory -i ./inventories/sample --list `


## Execute Playbooks - Source Gateways
```
$ ansible-playbook playbooks/gateway-preupgrade-analyzer.yml -i inventories/sample/hosts --vault-password-file vault-password-file.txt

$ ansible-playbook playbooks/gateway-database-export.yml -i inventories/sample/hosts --vault-password-file vault-password-file.txt

ROOT $ ansible-playbook playbooks/gateway-basic-backup.yml -i inventories/sample/hosts --vault-password-file vault-password-file.txt

ROOT $ ansible-playbook playbooks/gateway-restore-basic-backup.yml -i inventories/sample/hosts

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

```
$ ansible-playbook playbooks/gateway-autoprovision-nodes.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

ROOT $ ansible-playbook playbooks/gateway-database-replication.yml -i inventories/demo/hosts

$ ansible-playbook playbooks/gateway-database-import.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

$ ansible-playbook playbooks/gateway-database-schema-upgrade.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

$ ansible-playbook playbooks/gateway-install-license.yml -i inventories/demo/hosts --vault-password-file vault-password-file.txt

ROOT $ ansible-playbook playbooks/gateway-restore-basic-backup.yml -i inventories/demo/hosts

$ ansible-playbook playbooks/gateway-restart.yml -i inventories/demo/hosts

```


