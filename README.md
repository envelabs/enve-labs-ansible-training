# Ansible
`ansible` is an agentless automation tool that provides support for system configuration and deployments

### Control Machine Setup
in order to config the control machine required to orchestrate the ansible execution, `ansible` package is required.

#### ansible installation
```
sudo apt -y update
sudo apt install -y software-properties-common
sudo apt-add-repository -y ppa:ansible/ansible
sudo apt -y update
sudo apt install -y ansible
```
### Ansible `ad hoc` command execution

#### commands exec
check connectivity
```
ansible -i <inventory> -m ping <target-host>
```

check connectivity providing `ssh private-key`
```
ansible -i <inventory> --private-key=</path/to/key> -m ping <target-host>
```

exec ad hoc commands using the `shell` module
```
ansible -i <inventory> -m shell -a "ifconfig" <target-host>
```

### Ansible `playbooks` execution
plabook execution
```
ansible-playbook -i <inventory> <playbook> -e "host=<target-host>"
```

plabook execution with variables
```
ansible-playbook -i <inventory> <playbook> -e "<var>=<var-value>"
```

### Ansible `roles` creation
to be added

#### Handlers
if a task notifies a handler, then a subsequent task in the same play fails, then the whole playbook run will terminate and the handler will not be called. The workaround is to force the handler exec

```
# playbook with handlers

...
- meta: flush_handlers
...
```

#### Ansible Vault
Encrypt
```
ansible-vault encrypt <file-to-encrypt> --vault-password-file </path/to/vault-passwd-file>
```

Edit
```
ansible-vault edit <file-to-encrypt> --vault-password-file </path/to/vault-passwd-file>
```

Decrypt
```
ansible-vault decrypt <file-to-decrypt> --vault-password-file </path/to/vault-passwd-file>
```

Varfile format for <file_to_encrypt>.yml
```
---
<name_var_value>: |
  -----BEGIN RSA PRIVATE KEY-----
  MIIEogIBAAKCAQEA8uIMcbA07LJXt3TPpJfAlRaFu8MQYgFXibkafaMRGEBUHFb+
  PKyeJ6ksOzSXKjP1+W7z+593EktxKKoFSUCiHcPl5G/tHhMj8xnp8kf26WHnodc6
  pBEiTBB6czxg9wZE0jZ8DtHkQ7d1++p/8T2lP3x17chpcldwczOB+MjPsqfIW5
  ...
  M0Qmk7uqpYpKN5tFUYgXWKNhhYaGhhUKvJPkguu0Pexl0Px4+x+a42LnCX59JPND
  ehtVWez5f7TasdfivRfkn5tcNqEpTcpQ8pBzLMIuYe/GMtYSWNJXhetLtPMntYyK
  MH4ySw0qqXbqZcZAjZeWIV/UrtXQRlISpdqbqwIDAQABAoIBAG5epFMBRHuO62dV
```

### Ansible execution against AWS
```
AWS_PROFILE=<aws-profile>
ansible-playbook -i ec2.py <playbook> -e "host=<target-host>"
```

#### Ansible provisioning using `user_data`
inline shell
```
...
 tasks:
  - name: launch new EC2 instance
    ec2:
      key_name: "{{ ec2_keypair }}"
      user_data: |
        #!/bin/sh
        sudo apt-get install -y python
      volumes:
...
```

bash script
```
...
 tasks:
  - name: launch new EC2 instance
    ec2:
      key_name: "{{ ec2_keypair }}"
      group_id: "{{ ec2_security_group }}"
      instance_type: "{{ ec2_instance_type }}"
      image: "{{ ec2_image }}"
      vpc_subnet_id: "{{ ec2_subnet_ids | random }}"
      region: "{{ ec2_region }}"
      instance_tags: '{"Name":"{{ host }}", "id":"{{ host }}"}'
      assign_public_ip: yes
      wait: true
      count: 1
      user_data: "{{ lookup('file', 'base_provisioning.sh') }}"
      volumes:
...
```
