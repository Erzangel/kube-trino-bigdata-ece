# Lab Report

| Name              | Email                       |
| ---------------- | --------------------------- |
| Luka BIGOT       | luka.bigot@edu.ece.fr       |
| Cléa DEDUIT      | clea.deduit@edu.ece.fr      |
| Mathias SERICOLA | mathias.sericola@edu.ece.fr |

## Project description

`TODO`

## How did we implement it?

We have documented each step separately in the following files:

0. [Creating & provisioning VMs with Vagrant](./docs/0-vms.md)
1. Deploying Kubernetes with kubeadm, kubelet, kubectl
2. Deploying basic Kubernetes services (load balancer, Ingress)
3. Deploying MinIO to provide object storage
4. Deploying Trino to perform distributed computation

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