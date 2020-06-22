# Lab exercise
the repo contains a `Vagrantfile` that deploys a `master` VM and two `nodes` using `vagrant` as a VM config tool. The `master` VM is provisioned trough `inline` shell with Ansible in order to be used as a Control Machine against the `nodes` and execute different `ansible` commands

#### Prerequisites
Vagrant and VirtualBox must be installed and configured in the `host` machine

### Vagrant environment creation
stack deployment
```
vagrant up
```

list running virtual machines
```
vagrant status
Current machine states:

master                    running (virtualbox)
node1                     running (virtualbox)
node2                     running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

### Ansible inventory definition
in order to exec commands against the `targets` an `inventory` file is required where a `hostname` or a `group-name` should be defined to be used as `target`

```
[group-name]
hostname

```

To obtain the IPs used by the VMs it is needed to extract the IPs assigned to the `public_network` interface and then add the obtained value to the `inventory` file

From the `host` machine exec the following `VirtualBox` command
```
VBoxManage guestproperty get node1 /VirtualBox/GuestInfo/Net/1/V4/IP | awk {'print $2'}
192.168.0.126
```
or the `vagrant` command
```
vagrant ssh node1 -c "ip address show eth1 | grep 'inet ' | sed -e 's/^.*inet //' -e 's/\/.*$//'"
192.168.0.126
```

Add the IP to the `inventory` file as a parameter for the `ansible_host` connection variable
```
[masters]
master ansible_host=192.168.0.128 ansible_connection=ssh ansible_user=vagrant ansible_port=22 ansible_ssh_private_key_file=.vagrant/machines/master/virtualbox/private_key

[webservers]
node1 ansible_host=192.168.0.126 ansible_connection=ssh ansible_user=vagrant ansible_port=22 ansible_ssh_private_key_file=.vagrant/machines/node1/virtualbox/private_key

[dbservers]
node2 ansible_host=192.168.0.6 ansible_connection=ssh ansible_user=vagrant ansible_port=22 ansible_ssh_private_key_file=.vagrant/machines/node2/virtualbox/private_key

[targets:children]
webservers
dbservers

```

List all the `targets` defined in the `inventory` file
```
ansible -i inventory all --list-hosts
hosts (3):
	master
	node1
	node2
```

List all the `targets` defined in the `webservers` group in the `inventory` file
```
ansible -i inventory webservers --list-hosts
  hosts (1):
    node1
```

### Ansible `ad hoc` commands


exec a ping command against all the `targets` defined in the `inventory` file
```
ansible -i inventory all -m ping
node2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

master | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

node1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

exec a `date` command against one of the `targets` defined in the `inventory` file using the `command` module, which is the default module if no module name is defined
```
ansible -i inventory master -a "/bin/date"
master | CHANGED | rc=0 >>
Sun Jun 21 03:07:58 UTC 2020
```

extract the public IP from a `target` using the `shell` module, which is the required in order to concatenate commands
```
ansible -i inventory master -m shell -a "ip address show eth1 | grep 'inet ' | sed -e 's/^.*inet //' -e 's/\/.*$//'"
master | CHANGED | rc=0 >>
192.168.0.128
```
copy the `/etc/hosts` file from `host` to `master` VM using the `copy` module
```
ansible -i inventory master -m copy -a "src=/etc/hosts dest=/tmp/hosts"
master | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "076374ea2a4f32402314ce381b44d8e1fd414ccf",
    "dest": "/tmp/hosts",
    "gid": 1000,
    "group": "vagrant",
    "md5sum": "ac5e6ce18ed06af78d6f98986cb12297",
    "mode": "0664",
    "owner": "vagrant",
    "size": 280,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1592709739.3048599-48936-263686974850135/source",
    "state": "file",
    "uid": 1000
}
```

install apache2 package using `apt` modeule
```
ansible -i inventory node1 -m apt -a "name=apache2 state=present" --become
node1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "cache_update_time": 1592710090,
    "cache_updated": false,
...

"Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service -> /lib/systemd/system/apache2.service.",
"Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service -> /lib/systemd/system/apache-htcacheclean.service.",
"Processing triggers for libc-bin (2.27-3ubuntu1) ...",
"Processing triggers for systemd (237-3ubuntu10.25) ...",
"Processing triggers for ureadahead (0.100.0-21) ...",
"Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ..."
	]
}
```
### Ansible `playbooks` execution
[to be added]

### Ansible `roles`
[to be added]

### SSH keys
generate SSH private/public keys from `master`, in order to be added to  `node1` and `node2` under `/home/vagrant/.ssh/authorized_keys`
```
	ssh-keygen -t rsa
```
