# Ansible Collection - rbcapps_us.k3s_ansible

A no-nonsense Ansible collection to easily provision or upgrade a K3s cluster.

Handles edge cases, and has some significant built-in cluster planning logic, such as:
* When a node's role (server versus agent) changes in the configuration
* When the k3s version to be installed on a node is different than the version which is currently installed on the node (might fail if there is a version difference of more than one minor version, per the Kubernetes documentation)
* Adding new nodes to a cluster
* Automatically calculates the number of server nodes (can be overridden by setting a variable in the inventory) to an odd number, in order to establish quorum for the cluster state
* If there are at least 3 nodes in the cluster, the first 3 nodes will be server nodes, and the rest will be agents
* If there are fewer than 3 nodes in the cluster, the first node will be a server node, and the other node (if it exists) will be an agent node
* If a node needs to have k3s reinstalled on it (due to a change in k3s version or the node's role), the node will be automatically cordoned, drained, and removed from the cluster, k3s will be uninstalled from the node, the new k3s package (with the correct version) will be installed on the node with the correct role, and the node will be rejoined to the cluster under its new role
* Automatically installs Helm, Headlamp, Longhorn block storage, the CNI plugins, and Multus-CNI (each of these can be selectively installed or not installed, based on booleans which can be set from the calling class, or in the inventory)

## Testing on Virtual Machines in UTM on MacOS

See the [VMS.md](VMS.md) file for instructions to prepare three UTM VMs on your Mac for deploying K3s.

NOTE: Regardless of whether you are using VMs on your desktop, hosted VMs, or real hardware, the prerequisite users and configuration changes detailed in the above-referenced document are necessary.  Please pay careful attention to setting these up correctly before proceeding.

If you're not running on MacOS, you can use a different virtual machine package (VirtualBox, etc.) to create the VMs, but once the VMs are created and accessible from your laptop, the instructions are about the same.

## Test using the examples diretory

Change into the *examples* directory.  You will be working from there.
```sh
cd examples
```

Install Ansible:
```sh
brew install ansible
```

Install the required Galaxy module(s):
```sh
ansible-galaxy install -r requirements.yml
```

Create a file named *inventories/k3s-test-cluster/ansible-become-password.txt*, and put the password for the *ansible* user which you created when you set up the VMs.

NOTE: If you want to use the local copy of this collection, you can run the following command from the *examples* directory:
```sh
./install-local
```
This is also useful when making changes to the collection's playbooks or roles.  In that case, you can run the *install-local* before each playbook run, and that will ensure that you're always using the latest copy of the local playbooks and roles from the collection.

## Set the Inventory Environment Variables; Set up the K3s cluster

Set environment variables for the example inventory:
```sh
. ./set-ansible-inventory-k3s-test-cluster.sh
```

Run the Alive Check playbook to confirm that the nodes are alive:
```sh
ansible-playbook -i "$INVPATH" -u ansible --become-password-file "$BECOMEPWFILE" rbcapps_us.k3s_ansible.alive_check
```

Set up the K3s cluster:
```sh
ansible-playbook -i "$INVPATH" -u ansible --become-password-file "$BECOMEPWFILE" rbcapps_us.k3s_ansible.setup_k3s_cluster
```

At the end of the playbook, there should be a Headlamp token.  Point a browser to <http://k3s-test-01.local/headlamp>, copy and paste the token into the token field, and click *Authenticate*.  Click on *Workloads* -> *Pods* to see all of the pods which are running on the cluster.

You can also access the Longhorn dashboard at <http://k3s-test-01.local/longhorn>.  The login credentials are set in the `longhorn_ui_credentials` setting in roles/init/defaults/main.yml](roles/init/defaults/main.yml), and can be overridden in your inventory (or your calling role or playbook, if you're writing your own playbooks or roles).

## Creating a production project

Copy the *examples* directory to a new directory outside of this project.

Modify the requirements.yml to use a production tag of the *rbcapps_us.k3s_ansible* Galaxy collection.

Remove any non-production copy of the *rbcapps_us.k3s_ansible* collection, and reinstall the configured version:
```sh
rm -rf ~/.ansible/collections/ansible_collections/rbcapps_us/k3s_ansible
ansible-galaxy install -r requirements.yml
```

Create a file named *inventories/k3s-test-cluster/ansible-become-password.txt*, and put the password for the *ansible* user on your cluster.

## Summary of Playboks in the Collection

This collection contains the following playbooks:

* [rbcapps_us.k3s_ansible.setup_k3s_cluster](playbooks/setup_k3s_cluster.yml): Builds out a K3s cluster across the nodes (hosts) in the inventory.
* [rbcapps_us.k3s_ansible.headlamp_token](playbooks/headlamp_token.yml): Creates a new token for logging into the Headlamp dashboard.
* [rbcapps_us.k3s_ansible.uninstall_k3s](playbooks/uninstall_k3s.yml): Uninstalls k3s from all nodes (hosts) in the inventory.
* [rbcapps_us.k3s_ansible.alive_check](playbooks/alive_check.yml): Confirms that all nodes are alive and reachable.
* [rbcapps_us.k3s_ansible.reboot](playbooks/reboot.yml): Reboots all nodes.
* [rbcapps_us.k3s_ansible.shutdown](playbooks/shutdown.yml): Shuts down all nodes.

## Playbook Defaults

The default values for the roles in the playbook are stored in [roles/init/defaults/main.yml](roles/init/defaults/main.yml), and each customizable default is documented in [roles/init/meta/argument_specs.yml](roles/init/meta/argument_specs.yml).  You can override these values by setting them in the inventory, or, if you are calling roles directly from your own playbooks, by explicitly setting facts or passing vars to the roles which you call.

## Re-using Roles in your own Playbooks

If you need to write your own playbooks, as opposed to just using the playbooks which are provided in this collection, you can simply make a local copy of any of the above-referenced playbooks in your own project, and do your customizations.

## Developing this Project

Install `ansible-lint` on MacOS:
```sh
brew install ansible-lint
```

Validate playbooks and roles for lint errors:
```sh
./validate-lint
```
