# Ansible playbook and roles to create a simple Kubernetes cluster with kubeadm

## Goal

Provide an Ansible playbook that implements the steps described in [Installing Kubernetes on Linux with kubeadm](http://kubernetes.io/docs/getting-started-guides/kubeadm/)

## Assumptions

This playbook assumes: 

* You have access to 3+ Linux machines, with either CentOS 7 or Ubuntu 14.04 installed
* Full network connectivty exists between the machines, and the Ansible control machine (e.g. your computer)
* The machines have access to the Internet
* You are Ansible-knowledgable, can ssh into all the machines, and can sudo with no password prompt

## Configuration

1. Copy, rename, and edit the file INVENTORY-EXAMPLE
2. Enter the FQDN of the machine assigned the role of Kubernetes master in the ```[master]``` group.
3. Enter the FQDNs of the machines assigned the role of Kubernetes node in the ```[nodes]``` group.
4. Edit ```cluster_name``` (this will become the basename of a file used to store the ```admissions_token```)
5. Optionally uncomment and edit ```master_ip_address_configured``` in the ```[master:vars]``` section, if your master machine has multiple interfaces, and the default interface is NOT the interface you want the nodes to use to connect to the master.

After you have done this, you should be able to succesfully execute something like this:

```
    ansible -m ping -i YOUR-INVENTORY-FILE
```

And your master and node machines should respond.  Don't bother proceeding until this works!

## Run the playbook

When you are ready to proceed, run:

```
    ansible-playbook cluster.yml -i YOUR-INVENTORY-FILE
```

This should execute/implement all four installation steps in the aforementioned installation guide.

The guide provides examples you can run to test your cluster.

If you want to interact with your cluster via the kubectl command on your own machine (and why wouldn't you?), take note of the last note in the "Limitations" section of the guide:

```
There is not yet an easy way to generate a kubeconfig file which can be used to authenticate to the cluster remotely with kubectl on, for example, your workstation.
Workaround: copy the kubelet’s kubeconfig from the master: use scp root@<master>:/etc/kubernetes/admin.conf . and then e.g. kubectl --kubeconfig ./admin.conf get nodes from your workstation.
```

## Notes and Caveats

This playbook works for me, but is not extensively tested.
I have successfully run this to completion on a 3 machine Ubuntu setup, and only got as far as testing installation of the master for CentOS (I ran out of machines to test on!)

I don't yet understand much or anything about the Kubernetes pod network "Weave Net" the guide and this playbook installs.  Be forewarned!










