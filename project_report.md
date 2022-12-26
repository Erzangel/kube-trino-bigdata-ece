# Lab Report

| Name              | Email                       |
| ---------------- | --------------------------- |
| Luka BIGOT       | luka.bigot@edu.ece.fr       |
| Cléa DEDUIT      | clea.deduit@edu.ece.fr      |
| Mathias SERICOLA | mathias.sericola@edu.ece.fr |

## Project description

`TODO`

> Despite the commit history being slightly inconsistent, all commits have been done as a group and not by a single individual. **From start to finish, we have worked as a group**, and each member is aware of what has been done where, and for what purpose. The majority of commits from a single account is because we used a single SSH key to push/pull to and from GitHub inside the VM. We have tried to add co-authors as much as possible in the commit history regardless.

## Disclaimer about the host machine we used

As described in the `README.md`, for our host machine running the cluster of VMs, we used a VM with **very high specs** rented on a server of a friend of Luka Bigot.

However, because of this, and because we created a cluster of VMs on this VM with very high specs, we understand **it may be troublesome for evaluation to create the cluster yourself**.

We have done our best to provide screenshots for each category of the deployment of this distributed system in Kubernetes, so that you can see the results for yourself without running the cluster.

But because we were able to use a VM with such high specs (which was fun, to be honest), if you want to use it yourself to give a `vagrant up` a try in ideal conditions, do not hesitate to reach out to us! Contact any of us through our email addresses in the `README`.

## How did we implement it?

We have documented each step separately in the following files:

0. [Creating & provisioning VMs with Vagrant](./docs/0-vms.md)
1. [Deploying Kubernetes with kubeadm, kubelet, kubectl](./docs/1-deploying-kubernetes.md)
2. [Deploying basic Kubernetes services (CNI)](./docs/2-basic-kubernetes-services.md)
3. [Deploying MinIO to provide object storage](./docs/3-minio-object-storage.md)
4. [Deploying Trino to perform distributed computation](./docs/4-trino-processing.md)

## List of problems encountered

Here is a list of the major problems we have encountered during the project, and how we solved them.

### MinIO needs more than 4 hard disks in the cluster to work

We encountered an issue where we tried to set up MinIO using 3 nodes with 1 disk each. However, in order for MinIO to work (and specifically for Erasure-coding to be effective), we need **at least 4 hard disks in the cluster** as indicated in this [serverfault.com discussion](https://serverfault.com/questions/978644/deploy-minio-distributed-on-3-nodes-with-1-drive).

Two solutions are thus possible:

- Either add more disks per node (2 disks per node * 3 nodes = 6, OK)
- Or add one more worker node

Because we could afford the disk space and the RAM, we decided to apply both solutions: MinIO recommends 4 nodes for high-availability, and more volumes helps MinIO be more efficient as well.

## Links used to fix issues

- [Fixing kubeadm not starting because of kubelet](https://stackoverflow.com/questions/57648829/how-to-fix-timeout-at-waiting-for-the-kubelet-to-boot-up-the-control-plane-as-st)
- [Resizing disks on Linux (for our host VM)](https://www.msp360.com/resources/blog/linux-resize-partition/)
- [Debugging Kubelet service not starting on machines](https://serverfault.com/questions/877136/debug-kubelet-not-starting)