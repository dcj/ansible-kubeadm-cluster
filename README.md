# Ansible playbook and roles to create/manage a simple Kubernetes cluster with kubeadm

## Goal

Provide an Ansible playbook that implements the steps described in [Installing Kubernetes on Linux with kubeadm](http://kubernetes.io/docs/getting-started-guides/kubeadm/)

## Please Note

Newly revised for Kubernetes/kubeadm 1.6.1

Release 1.6 broke kubeadm, and there was an interim release here that worked around that, but with the new 1.6.1 release things seem to be working better.

I haven't tried/used CentOS 7 in months, and never got it to work.  My current focus/testing is on Ubuntu 16.04, and this is working well there.

The structure of these playbooks has changed significantly since their initial release.
Specifically the structure/hierarchy of the Ansible Inventory file has changed, and the playbooks now
require this new structure.

In brief, there used to be one inventory file per cluster.
Now there can be many clusters defined in the inventory, and the group structure/hierarchy defined here is required.
The cluster\_name must be passed into the playbook (e.g. --extra-vars "cluster_name=foo" )

There are important advantages to this breaking change, specifically it is now much easier to use these playbooks with your existing
inventory (assuming you add the necessary sub-groups), and now it is straightforward to manage/operate on multiple clusters.

## Assumptions

These playbooks assume:

* You have access to 3+ Linux machines, with Ubuntu 16.04 installed (CentOS 7 support currently out of date and broken)
* Full network connectivty exists between the machines, and the Ansible control machine (e.g. your computer)
* The machines have access to the Internet
* You are Ansible-knowledgable, can ssh into all the machines, and can sudo with no password prompt
* Make sure your machines are time-synchronized, and that you understand their firewall configuration and status

## Ansible Inventory Structure and Requirements

1. Define an Ansible inventory group for each Kubernetes cluster you wish to create/manage, e.g. ```k8s_test```
2. Define two Ansible child groups under each cluster group, with names ```_master``` and ```_node```, for example ```k8s_test_master``` and ```k8s_test_node```
3. List the FQDN of the machine assigned the role of Kubernetes master in the ```[cluster_name_master]``` group.
3. List the FQDNs of the machines assigned the role of Kubernetes node in the ```[cluster_name_node]``` group.
5. Optionally add the variable ```master_ip_address_configured``` in the ```[cluster_name_master:vars]``` section, if your master machine has multiple interfaces, and the default interface is NOT the interface you want the nodes to use to connect to the master.

A sample Inventory file is included as ```cfg/INVENTORY-EXAMPLE```, but if you have/use an existing Ansible inventory, it is a lot easier to just add the structure described above to your existing inventory and use that, IMHO.

After you have done this, you should be able to succesfully execute something like this:

```
    ansible -m ping -e cluster_name=k8s_test
```

And your master and node machines should respond.  

Then test that you can operate on each of the child groups independently:

```
    ansible -m ping -e cluster_name=k8s_test_master
```

and

```
    ansible -m ping -e cluster_name=k8s_test_node
```

Don't bother proceeding until all the above work!

## Run the playbook

When you are ready to proceed, run something like:

```
    ansible-playbook cluster-create.yml -e cluster_name=k8s_test
```

This should execute/implement all four installation steps in the aforementioned installation guide.

The guide then provides examples you can run to test your cluster.

If you want to interact with your cluster via the kubectl command on your own machine (and why wouldn't you?), take note of the last note in the "Limitations" section of the guide:

> There is not yet an easy way to generate a kubeconfig file which can be used to authenticate to the cluster remotely with kubectl on, 
> for example, your workstation. Workaround: copy the kubelet’s kubeconfig from the master: use 

>   `scp root@<master>:/etc/kubernetes/admin.conf . `

> and then e.g. 

>   `kubectl --kubeconfig ./admin.conf get nodes` 

> from your workstation.


The playbook retrieves the admin.conf file, and stores it locally as ```./cfg/cluster_name/admin.conf``` to facilitate remote kubectl access.

## Additional Playbooks

```
    ansible-playbook cluster-infra.yml -e cluster_name=k8s_test
```

Starts the Dashboard and WeaveScope on the cluster.

```
    ansible-playbook cluster-destroy.yml -e cluster_name=k8s_test
```

Completely destroys your cluster, with no backups. Don't run this unless that is what you really want!

## Idempotency

* I belive I have made admission token generation idempotent. Generated tokens are stored in ```./cfg/cluster_name/token.yml```, and reused on subsequent playbook runs
* I'm not sure how to know that the init and join operations have successfully completed, I've tried to base it on files/directories that are created, but not yet certain that is correct.
* It seems like re-issuing the ```kubectl apply -f https://git.io/weave-kube``` is harmless, but again, I'm not certain...

## Notes and Caveats

* This playbook is under active development, but is not extensively tested.
* I have successfully run this to completion on a 3 machine Ubuntu setup, it basically worked the first time.
* I haven't yet succeeded in getting a cluster working perfectly on Centos 7. As of 2017-04-03 CentOS 7 support is completely out of date and currently unsupported.
* I don't yet understand much or anything about the Kubernetes pod network "Weave Net" the guide and this playbook installs.  Be forewarned!

## TODO

* Migrate token generation to use the new ```kubeadm token generate``` feature
* Figure out how to get and use JSON output of kubectl commands issued by these playbooks. This should be straightfoward/trival but I've run into issues with certain invoications of kubectl returning invalid JSON (it returns multiple objects at the top-level, not in an array...)
* Fix/improve/update CentOS 7 support

## Acknowlegements

* Huge kudos to the authors of kubeadm and [its getting started guide](http://kubernetes.io/docs/getting-started-guides/kubeadm/)
* Joe Beda [provided the code to generate tokens, and how to feed a token into kubeadm init](https://github.com/upmc-enterprises/kubeadm-aws/issues/1)
* @marun pointed me to the documentation about how to access the master remotely via kubectl
* @VAdamec forked this repo, and made some improvements, several of which I have incorporated here.

## Contributing

Pull requests and issues are welcome.












