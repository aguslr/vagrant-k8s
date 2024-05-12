[aguslr/vagrant-k8s][1]
=======================


This repository contains the configuration files necessary to orchestrate a
[Kubernetes][2] cluster using [Vagrant][3] and provision it with [Ansible][4].
It works both with [VirtualBox][6] and [Libvirt][5] boxes.


Installation
------------

Before anything, [Vagrant][3] and either [libvirt][5] or [VirtualBox][6] should
be installed:

- For APT based distributions:

      sudo apt install vagrant vagrant-libvirt

- For RPM based distributions:

      sudo dnf install vagrant vagrant-libvirt

- For Windows and macOS, refer to [Vagrant][7]'s and [VirtualBox][8]'s download
  pages.


### Set-up ###

First of all, we have to clone the repository:

    git clone https://github.com/aguslr/vagrant-k8s && cd vagrant-k8s

Afterwards, to orchestrate a cluster, we follow these steps:

1. Setup the VMs with this command:

       vagrant up

2. Once the VMs are up, connect to the control plane:

       vagrant ssh master -- kubectl get nodes -o wide

Alternatively, we can attach the VMs to a physical interface so they are
reachable from any machine in the network:

1. Assign the interface to a variable and setup the VMs:

       BRIDGE_IFACE=br0 vagrant up

2. Access the Dashboard UI by connecting to the URL that is displayed in a
   post-up message along with the token.


#### Managing Kubernetes ####

To use the [Kubernetes client][9] to interact with the cluster from our local
machine, we must prepare the environment:

1. Copy the configuration locally:

       vagrant ssh master -- cat .kube/config > ${KUBECONFIG:-$HOME/.kube/config}

2. Now we can run `kubectl` commands:

       kubectl get nodes -o wide


Configuration
-------------

Everything can be configured using environment variables:

| Variable              | Function                              | Default             |
| :-------------------- | :------------------------------------ | :------------------ |
| `BRIDGE_IFACE`        | Network interface to attach VMs to    | empty               |
| `K8S_MAC_ADDRESS`     | MAC address for master node           | `525400000a00`      |
| `K8S_MASTER_CPUS`     | Number of CPUs for master node        | `4`                 |
| `K8S_MASTER_MEMORY`   | Amount of memory for master node      | `2048`              |
| `K8S_NODES_COUNT`     | Number of worker nodes                | `2`                 |
| `K8S_NODE_CPUS`       | Number of CPUs for each worker node   | `2`                 |
| `K8S_NODE_MEMORY`     | Amount of memory for each worker node | `2048`              |
| `LIBVIRT_DEFAULT_URI` | URI for libvirt daemon to connect to  | `qemu:///system`    |
| `VAGRANT_BOX`         | Remote image to use as base for VMs   | `debian/bookworm64` |

For example, to orchestrate a cluster with nodes running Debian 12 attached to
the network interface `eth0`, we do:

    BRIDGE_IFACE=eth0 VAGRANT_BOX=generic/debian12 vagrant up


### Supported boxes ###

The following official boxes have been tested:

- [`almalinux/9`](https://app.vagrantup.com/almalinux/boxes/9) (libvirt,
  virtualbox)
- [`debian/bookworm64`](https://app.vagrantup.com/debian/boxes/bookworm64)
  (libvirt, virtualbox)
- [`rockylinux/9`](https://app.vagrantup.com/rockylinux/boxes/9) (libvirt,
  virtualbox)

Alternative, these non-official boxes have been tested:

- [`generic/alma9`](https://app.vagrantup.com/generic/boxes/alma9) (libvirt,
  virtualbox)
- [`generic/debian12`](https://app.vagrantup.com/generic/boxes/debian12)
  (libvirt, virtualbox)
- [`generic/ubuntu2204`](https://app.vagrantup.com/generic/boxes/ubuntu2204)
  (libvirt, virtualbox)


References
----------

- <https://developer.hashicorp.com/vagrant/docs/provisioning/ansible>
- <https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_common>
- <https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_local>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/first_found_lookup.html>
- <https://docs.ansible.com/ansible/latest/collections/kubernetes/core/k8s_module.html#ansible-collections-kubernetes-core-k8s-module>
- <https://docs.ansible.com/ansible/latest/collections_guide/collections_installing.html#install-multiple-collections-with-a-requirements-file>
- <https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html#with-sequence>
- <https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html>
- <https://github.com/cri-o/packaging/blob/main/README.md>
- <https://github.com/justmeandopensource/kubernetes>
- <https://github.com/kubernetes/dashboard/blob/v2.7.0/docs/user/access-control/creating-sample-user.md>
- <https://github.com/kubernetes/dashboard/blob/v2.7.0/docs/user/accessing-dashboard/README.md>
- <https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/>
- <https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/>
- <https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o>
- <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>
- <https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/>
- <https://libvirt.org/uri.html>
- <https://vagrant-libvirt.github.io/vagrant-libvirt/configuration.html>


[1]: https://github.com/aguslr/vagrant-k8s
[2]: https://kubernetes.io/
[3]: https://www.vagrantup.com/
[4]: https://www.ansible.com/
[5]: https://vagrant-libvirt.github.io/vagrant-libvirt/
[6]: https://developer.hashicorp.com/vagrant/docs/providers/virtualbox
[7]: https://developer.hashicorp.com/vagrant/install
[8]: https://www.virtualbox.org/wiki/Downloads
[9]: https://kubernetes.io/docs/reference/kubectl/
