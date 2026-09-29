# Ansible Collection - roncemer.k3s_ansible

A no-nonsense Ansible collection to easily provision or upgrade a K3s cluster.

Handles edge cases, and has some significant built-in cluster planning logic, such as:
* When a node's role (server versus agent) changes in the configuration
* When the k3s version to be installed on a node is different than the version which is currently installed on the node (might fail if there is a version difference of more than one minor version, per the Kubernetes documentation)
* Adding new nodes to a cluster
* Automatically calculates the number of server nodes (can be overridden by setting a variable in the inventory) to an odd number, in order to establish quorum for the cluster state
* If there are at least 3 nodes in the cluster, the first 3 nodes will be server nodes, and the rest will be workers
* If there are fewer than 3 nodes in the cluster, the first node will be a server node, and the other node (if it exists) will be a worker node
* If a node needs to have k3s reinstalled on it (due to a change in k3s version or the node's role), the node will be automatically cordoned, drained, and removed from the cluster, k3s will be uninstalled from the node, the new k3s package (with the correct version) will be installed on the node with the correct role, and the node will be rejoined to the cluster under its new role
* Automatically installs Helm 4.x on every server node in the cluster
* Automatically installs the Headlamp dashboard on the cluster

## Testing on Virtual Machines in UTM on MacOS

See the [VMS.md](VMS.md) file for instructions to prepare three UTM VMs on your Mac for deploying K3s.

NOTE: Regardless of whether you are using VMs on your desktop, hosted VMs, or real hardware, the prerequisite users and configuration changes detailed in the above-referenced document are necessary.  Please pay careful attention to setting these up correctly before proceeding.

If you're not running on MacOS, you can use a different virtual machine package (VirtualBox, etc.) to create the VMs, but once the VMs are created and accessible from your laptop, the instructions are about the same.

## Test using the examples diretory

Change into the *examples* directory.  You will be working from there.
```sh
cd examples
```

Install Ansible and the required Galaxy module(s):
```sh
brew install ansible
ansible-galaxy install -r requirements.yml
```

Create a file named *inventories/k3s-test-cluster/ansible-become-password.txt*, and put the password for the *ansible* user which you created when you set up the VMs.

NOTE: If you want to use the local copy of this collection, you can run the following command from the *examples* directory:
```sh
./install-local
```
This is also useful when making changes to the collection's playbooks or roles.  In that case, you can run the *install-local* before each playbook run, and that will ensure that you're always using the latest copy of the local playbooks and roles from the collection.

## Set the Inventory Environment Variables; Set up the K3s cluster

```sh
. ./set-ansible-inventory-k3s-test-cluster.sh
ansible-playbook -i "$INVPATH" -u ansible --become-password-file "$BECOMEPWFILE" roncemer.k3s_ansible.setup_k3s_cluster
```

At the end of the playbook, there should be a Headlamp token.  Point a browser to <http://k3s-test-01.local/headlamp>, copy and paste the token into the token field, and click *Authenticate*.  Click on *Workloads* -> *Pods* to see all of the pods which are running on the cluster.

## Creating a production project

Copy the *examples* directory to a new directory outside of this project.

Modify the requirements.yml to use a production tag of the *roncemer.k3s_ansible* Galaxy collection.

Remove any non-production copy of the *roncemer.k3s_ansible* collection, and reinstall the configured version:
```sh
rm -rf ~/.ansible/collections/ansible_collections/roncemer/k3s_ansible
brew install ansible  # (if not already installed)
ansible-galaxy install -r requirements.yml
```

Create a file named *inventories/k3s-test-cluster/ansible-become-password.txt*, and put the password for the *ansible* user on your cluster.

## Other useful playbooks

If you need to get another Headlamp token:
```sh
ansible-playbook -i "$INVPATH" -u ansible --become-password-file "$BECOMEPWFILE" roncemer.k3s_ansible.headlamp_token
```

To completely uninstall k3s from all nodes in the cluster, and delete the k3s user and group:
```sh
ansible-playbook -i "$INVPATH" -u ansible --become-password-file "$BECOMEPWFILE" roncemer.k3s_ansible.uninstall_k3s
```
