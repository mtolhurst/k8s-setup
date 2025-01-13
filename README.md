# K8s cluster manager

## TODO
- [ ] Start deploying other applications
- [ ] Hubble for network visibility (could help writing cilium rules)
- [ ] Begin adding cilium rules
- [ ] Tone down resource requests
- [ ] Consider swapping influxdb for prometheus to get PG monitoring

## Node pre-reqs
* This assumes a base debian install  (headless is fine)
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

```bash
ansible-playbook -i inventory --key-file <PATH_TO_PRIVATE_KEY> -u <USERNAME> create_cluster.yml
```

This should create the control plane, create and join worker nodes, then install a number of services listed below

## Included services
### Networking
* Currently, I'm using cilium as the CNI
  * Using kube-proxy replacement
  * Using l2 announcements + LBIPAM to expose services to the external network
  * Eventually I'd like to add default-deny network policies, only enabling traffic that's required
* Important note about home network
  * Most devices are on 192.168.1.0/24
  * There is a bridge to 10.0.0.0/8 though, allowing applications to be exposed there
  * This allows us to setup an lb IPAM pool somewhere in this range


### Storage
* Longhorn - distributed block storage
  * Running on `/longhorn` behind the reverse proxy
  * Provides PV/PVC to the cluster
  * If space starts running out here, can either expand disks, or get a NAS for iscsi

### Monitoring
* Influxdb for storing time-series data
  * Setup with hardcoded credentials in the config - ideally these would be auto-generated
* Grafana for graphing metrics
  * It is not currently automatically configured from this repo
  * I followed https://www.influxdata.com/blog/deploying-influxdb-telegraf-to-monitor-kubernetes/ to set it up
* Telegraf for metric collection
  * This is configured as a daemonset to automatically ship metrics to influxDB
  * It is not exposed outside the cluster