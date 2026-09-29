# Ansible Collection - roncemer.k3s_ansible

An Ansible collection to easily provision or upgrade a K3s cluster

## Testing on Virtual Machines in UTM on MacOS

See the [VMS.md](VMS.md) file for instructions to prepare three UTM VMs on your Mac for deploying K3s.

If you're not running on MacOS, you can use a different virtual machine package (VirtualBox, etc.) to create the VMs, but once the VMs are created and accessible from your laptop, the instructions are about the same.

## Create a basic project

Start in a new, empty directory.

Create a file named *requirements.yml*, and copy the following contents into it (change version to the release tag if you want to use a specific release, which you always should do in production):
```yaml
collections:
  - name: https://github.com/roncemer/k3s-ansible
    type: git
    version: main
```

Install Ansible and the required Galaxy module(s).

```sh
brew install ansible
ansible-galaxy install -r requirements.yml
```

## Create an inventory

Run the following command to create the inventory directory:

```sh
mkdir -p inventories/k3s-test-cluster/group_vars
```

Create a file named *inventories/k3s-test-cluster/hosts*, and add the following contents into it:
```text
[all_hosts]
k3s-test-01.local 
k3s-test-02.local 
k3s-test-03.local 
```

Create a file named *inventories/k3s-test-cluster/group_vars/all.yml*, and add the following contents into it:
```text
# This cluster uses sudo.ws.  Without the next line, all tasks with "become: true" will fail.
ansible_become_exe: /usr/bin/sudo.ws
```

Create a file named *inventories/k3s-test-cluster/ansible-become-password.txt*, and put the password for the *ansible* user which you created when you set up the VMs.

Create a file named *set-ansible-inventory-k3s-test-cluster.sh*, and copy the following contents into it:
```text
INVPATH="inventories/k3s-test-cluster"
export INVPATH
BECOMEPWFILE=inventories/k3s-test-cluster/ansible-become-password.txt
export BECOMEPWFILE
```

## Set the Inventory Environment Variables; Set up the K3s cluster

```sh
. ./set-ansible-inventory-k3s-test-cluster.sh
ansible-playbook -i "$INVPATH" -u ansible --become-password-file "$BECOMEPWFILE" roncemer.k3s_ansible.setup_k3s_cluster
```

At the end of the playbook, there should be a Headlamp token.  Point a browser to <http://k3s-test-01.local/headlamp>, copy and paste the token into the token field, and click *Authenticate*.  Click on *Workloads* -> *Pods* to see all of the pods which are running on the cluster.

## Useful Playbooks

If you need to get another Headlamp token:
```sh
ansible-playbook -i "$INVPATH" -u ansible --become-password-file "$BECOMEPWFILE" roncemer.k3s_ansible..headlamp_token
```
