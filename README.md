### What to change
```bash
# example when deploying orion cluster.
Change the user in ansible.cfg
Adapt variables in the inventory file ./group_vars/inventories/orion/orion.yml
```

### Test Hosts
```bash
export ANSIBLE_LOG_PATH=./logs/sys_info_$(date "+%Y-%m-%d_%H%M%S").log
ansible-playbook -i ./inventories/orion/orion.ini ./playbooks/check-servers.yml --extra-vars "@./group_vars/all.yml"
```

---
### Install RKE2
```bash
#Management cluster
export ANSIBLE_LOG_PATH=./logs/install_mgt_$(date "+%Y-%m-%d_%H%M%S").log
ansible-playbook -i ./inventories/orion/orion.ini cluster.yml --extra-vars "@./group_vars/rke2-config.yml"
```

---
### Uninstall RKE2
```bash
export ANSIBLE_LOG_PATH=./logs/uninstall_$(date "+%Y-%m-%d_%H%M%S").log 
ansible-playbook -i ./inventories/orion/orion.ini ./playbooks/uninstall-cluster.yml
this will trigger the following script
Script > `/usr/local/bin/rke2-uninstall.sh`
```

---
### Server Config
Server config consist of configuring DNS, Firewall, Ntables . . . . 
```bash
export ANSIBLE_LOG_PATH=./logs/server-config_$(date "+%Y-%m-%d_%H%M%S").log
ansible-playbook -i ./inventories/orion/orion.ini ./playbooks/server-config.yml --extra-vars "@./group_vars/server-config.yml"
```

---
### Export Kubeconfig to Vault

#### Load Env Vars
```bash
export VAULT_ROLE_ID=""
export VAULT_SECRET_ID=""
export VAULT_ADDR=""
```

#### Run Playbook
```bash
export ANSIBLE_LOG_PATH=./logs/cluster_$(date "+%Y-%m-%d_%H%M%S").log
ansible-playbook -i ./inventories/zme-orion/management/orion.ini ./playbooks/vault-exporter.yml --extra-vars "@./group_vars/all.yml" --extra-vars "@./inventories/zme-orion/management/management.yml"
```

---
### DEBUG
When running with systemd, logs are sent to journald and can be viewed using `journalctl -u rke2-server` or `journalctl -u rke2-agent`. Some systemd configurations may also write combined logs to `/var/log/syslog`, in which case the rke2 logs will also be available there.

The Containerd logs are written to `/var/lib/rancher/rke2/agent/containerd/containerd.log`.

The kubelet logs are written to `/var/lib/rancher/rke2/agent/logs/kubelet.log`.

#### Stop RKE2 Service
Users observed the following behavior:
When running systemctl stop `rke2-server` on a master node or systemctl stop `rke2-agent` on a worker node, the service enters a failed state.
Despite the service being marked as "failed," some processes such as containerd and pods continue to run.
Attempting to kill the main process PID directly does not cleanly stop the related components.

To fully and cleanly stop all RKE2-related processes and containers, including `containerd`, use the `/usr/local/bin/rke2-killall.sh` script. This script is designed to:
Terminate all RKE2 processes
Stop containerd and associated workloads
Prevent residual resource conflicts when restarting or upgrading

---
### CRICTL Config
Config file
```bash
/etc/crictl.yaml
```

---
### Kubeconfig
Preconfigured KUBECONFIG can be found in the following path
```bash
#if your cluster is mgt
/etc/rancher/rke2/mgt/mgt.kubeconfig

#if you cluster is an ACR, for example NIM
/etc/rancher/rke2/nim/nim.kubeconfig
```
