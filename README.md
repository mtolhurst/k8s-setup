# K8s cluster manager

## Node pre-reqs
* This assumes a debian install
* Static IP configured on each node
* Swap disabled on all nodes
* SSH access to each node, with a user with sudo access
    * It is recommended this is via ssh keys

## Setup
TODO replace this with docker

### Create a python virtualenv
```sh
python -m venv -venv
source .venv/bin/activate
```

### Install required packages

```
# inside the virtualenv
pip install -r requirements.txt

ansible-galaxy collection install kubernetes.core
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.general
```

### Setup inventory
Add entries to `inventory/hosts.yml` for each node. 

### Test connection
```
ansible-playbook -i ./inventory ping_test.yml  
```

## Create the cluster

### Create the first control plane node
