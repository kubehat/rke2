### Set Env
export TARGET_ENV="orbit"
export ANSIBLE_INVENTORY="mgt"

### What to change
```bash
# example when deploying orion cluster.
Change the user in ansible.cfg
Adapt variables in the var file ./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.yml
Adapt Inventory hosts in the inventory file ./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.ini
```

### Test Hosts
```bash
ansible-playbook -i ./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.ini ./playbooks/check-servers.yml --extra-vars "@./group_vars/all.yml"
```

---
### Install RKE2
```bash
ansible-playbook -i ./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.ini cluster.yml --extra-vars "@./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.yml" --extra-vars "@./group_vars/server-config.yml"
```

---
### Uninstall RKE2
```bash
ansible-playbook -i ./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.ini ./playbooks/uninstall-rke2.yml --extra-vars "@./group_vars/all.yml" --extra-vars "@./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.yml"
```
this will trigger the following script
Script > `/usr/local/bin/rke2-uninstall.sh`


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
ansible-playbook cluster.yml --tags vault -i ./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.ini --extra-vars "@./inventories/$TARGET_ENV/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.yml"
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
/etc/rancher/rke2/$ANSIBLE_INVENTORY/$ANSIBLE_INVENTORY.kubeconfig
