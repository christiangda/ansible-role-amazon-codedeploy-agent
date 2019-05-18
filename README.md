# Ansible Role: christiangda.amazon_codedeploy_agent

[![Build Status](https://travis-ci.org/christiangda/ansible-role-amazon-codedeploy-agent.svg?branch=master)](https://travis-ci.org/christiangda/ansible-role-amazon-codedeploy-agent)
[![Ansible Role](https://img.shields.io/ansible/role/39191.svg)](https://galaxy.ansible.com/christiangda/amazon_codedeploy_agent)

This role [Install AWS CodeDeploy Agent](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)

## Requirements

This role work on RedHat, CentOS, Amazon Linux, Debian and Ubuntu distributions

* RedHat
  * 7
* CentOS
  * 7
* Amazon Linux
  * 1
  * 2
* Ubuntu
  * 14.04
  * 16.04
  * 18.10
* Debian
  * buster
  * jessie
  * sid
  * stretch

## Role Variables

```yaml
# posible values:
# - "{{ lookup('file', 'files/your-cloudwatch-config.json') | from_json }}" where your-cloudwatch-config.json is your custom
#   configuration file according to docs reference.
# - "{{ lookup('file', 'files/your-cloudwatch-config.yaml') | from_yaml }}" where your-cloudwatch-config.yaml is your custom
#   configuration file according to docs reference.
#
# Also is possible to define inline configuration as YAML
#    cwa_conf_json_file_content:
#      metrics:
#        append_dimensions:
#          AutoScalingGroupName: "${!aws:AutoScalingGroupName}"
#          ImageId: "${aws:ImageId}"
#          InstanceId: "${aws:InstanceId}"
#          InstanceType: "${aws:InstanceType}"
#        metrics_collected:
#          mem:
#            measurement:
#              - mem_used_percent
#          swap:
#            measurement:
#              - swap_used_percent
#
# reference: https://docs.aws.amazon.com/es_es/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html
# default value: ""
# notes:
# * when is empty the role deploy a custom json configuration via template
cwa_conf_json_file_content: ""
```

## Dependencies

* [christiangda.epel_repo](https://galaxy.ansible.com/christiangda/epel_repo) imported dynamically in the code when is used with Red Hat family, so you don't need to add this to your ansible playbook.

## Example Playbook

### RedHat/CentOS, Ubuntu and Debian

**Reading config file from JSON configuration file**

```yaml
- hosts: servers
    gather_facts: True
    roles:
    - role: christiangda.amazon_codedeploy_agent
        vars:
            cwa_agent_mode: "ec2"
            cwa_conf_json_file_content: "{{ lookup('file', 'files/CloudWatch.json') | from_json }}"
```

**Reading config file from YAML configuration file**

```yaml
- hosts: servers
    gather_facts: True
    roles:
    - role: christiangda.amazon_codedeploy_agent
        vars:
            cwa_agent_mode: "ec2"
            cwa_conf_json_file_content: "{{ lookup('file', 'files/CloudWatch.yaml') | from_yaml }}"
```

###  Amazon Linux 1/2 (my-playbook.yml)

```yaml
- hosts: all
    gather_facts: True
    become: true
    become_user: root
    become_method: sudo
    remote_user: ec2-user
    roles:
    - role: christiangda.amazon_codedeploy_agent
        vars:
            cwa_agent_mode: "ec2"
            cwa_conf_json_file_content: "{{ lookup('file', 'files/CloudWatch.json') | from_json }}"
```

**Inventory file sample (inventory)**

```ini
[all]
10.14.x.y
10.14.v.z

[amazon-1]
10.14.x.y

[amazon-2]
10.14.v.z
```

**How to used it**

```bash
ansible-playbook my-playbook.yml \
    --inventory inventory \
    --private-key [~/location of my key.pem] \
    --become \
    --become-user=ec2-user \
    --user ec2-user
```

### Variables examples:

```yaml
#cwa_conf_json_file_content: "{{ lookup('file', 'files/CloudWatch.json') | from_json }}"
cwa_conf_json_file_content: "{{ lookup('file', 'files/CloudWatch.yaml') | from_yaml }}"

cwa_agent_mode: "onPremise"
cwa_aws_region: "eu-west-1"

# You should use ansible-vault to provision it
# Example:
# ansible-vault encrypt_string --ask-vault-pass --name 'cwa_access_key' 'AKIAIOSFODNN7EXAMPLE'
# ansible-vault encrypt_string --ask-vault-pass --name 'cwa_secret_key' 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY'
cwa_access_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30376338613338326663373366303234623665633339303338613463313564633832363237306137
          3362643039616631323339383332306536333962346133310a383265376665316235653261616136
          61306566623531356263346439633761633830323636646236373736353530396134636536666532
          3939636433636364310a316639366139366566623337623536346661633339343766323936346336
          65333035366635396138656132643262626438333961326266396466626464643766
cwa_secret_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          65643230613939303737336632346432393234616437383532386139616364316233333933643735
          6633636261383163323362623562323333323533336564310a323030343431366135343035326635
          33623161376634643939636464306139386662303034616531346632303039643238373834616266
          3064623232373233610a346432646565396235316631626137653731376365333531323866626665
          62656638623330643539653763636364363738653932653831316238633939356462653636633463
          6130613761633565616533633332376565373062396565396261
```

## Development / Contributing

This role is tested using [Molecule](https://molecule.readthedocs.io/en/latest/) and was developed using
[Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)

**Prepare your environment**

Python 3

```bash
mkdir ansible-roles
cd ansible-roles/

python3 -m venv venv
source venv/bin/activate
pip install pip --upgrade
pip install ansible
pip install selinux
pip install docker
pip install pytest
pip install pytest-mock
pip install pylint
pip install rope
pip install autopep8
pip install yamllint
pip install molecule
pip install molecule[vagrant]
```

**Clone the role repository and create symbolic link**

```bash
git clone https://github.com/christiangda/ansible-role-amazon-codedeploy-agent.git
ln -s ansible-role-amazon-codedeploy-agent christiangda.amazon_codedeploy_agent
cd christiangda.amazon_codedeploy_agent
```

**Execute the test**

Using docker in local

```bash
molecule test [--scenario-name default]
```

Using vagrant in local

```bash
molecule create --scenario-name vagrant
molecule converge --scenario-name vagrant
molecule verify --scenario-name vagrant
```

or

```bash
molecule test --scenario-name vagrant
```

**Additionally if you want to test it using VMs, I have a very nice [ansible-playground project](https://github.com/christiangda/ansible-playground) that use Vagrant and VirtualBox, try it!.**

## License

This module is released under the GNU General Public License Version 3:

* [http://www.gnu.org/licenses/gpl-3.0-standalone.html](http://www.gnu.org/licenses/gpl-3.0-standalone.html)

## Author Information

* [Christian González Di Antonio](https://github.com/christiangda)
