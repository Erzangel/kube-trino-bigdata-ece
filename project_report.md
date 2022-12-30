# Lab Report

| Name              | Email                       |
| ---------------- | --------------------------- |
| Luka BIGOT       | luka.bigot@edu.ece.fr       |
| Cléa DEDUIT      | clea.deduit@edu.ece.fr      |
| Mathias SERICOLA | mathias.sericola@edu.ece.fr |

## Project description

Our goal for this project was to **deploy Trino on Kubernetes**, by following four main steps:

- Creating the Virtual Machines supporting the cluster ourselves, and automating all deployment;
- Creating the cluster from scratch, through `kubeadm` and not through `minikube`, in a way that would be as close to a real production-grade cluster;
- Deploying object storage to define a sink from where to consume data from (and Luka conveniently has experience with it!);
- Deploying Trino on the cluster.

All in all, this was mainly about **deploying a Kubernetes infrastructure** and **providing it with deployments enabling basic Data Analytics work**.

> Despite the commit history being slightly uneven, all commits have been done as a group and not by a single individual. **From start to finish, we have worked as a group**, and each member is aware of what has been done where, and for what purpose. The majority of commits from a single account is because we used a single SSH key to push/pull to and from GitHub inside the VM. We have tried to add co-authors as much as possible in the commit history regardless.

## Short summary of what we have tried and done

- We created VMs with Vagrant and automated their provisioning with Ansible;
- We deployed Kubernetes on those VMs in 1 day;
- We deployed MinIO and Trino in the cluster. Trino worked fine, but MinIO was difficult to set up;
- We have figured out multiple ways to debug a Kubernetes cluster, as described in the [section describing issues we ran into](#list-of-problems-encountered). In particular, it includes how to debug a Kubernetes cluster.

Because we spent a lot of time fixing the MinIO deployment, and despite wishing to cover an example use case with Trino, we have not been able to.

However, creating the cluster in itself was already a big ordeal, and we are proud to have made **our own Kubernetes cluster for Data Analytics** using the production-grade tool `kubeadm`.

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

Be sure to check out each documentation file linked just above, as it contains extensive notes about our understanding of each technology! It also contains the steps we performed to implement a given step.

## List of problems encountered

Here is a list of the major problems we have encountered during the project, and how we solved them.

**Most of those issues were related to object storage with MinIO** Resolving them has been the most time-consuming part of the project, and we have given this **more priority**, as having a fully-functional distributed system is an objective we set ourselves.

### MinIO needs more than 4 hard disks in the cluster to work

We encountered an issue where we tried to set up MinIO using 3 nodes with 1 disk each. However, in order for MinIO to work (and specifically for Erasure-coding to be effective), we need **at least 4 hard disks in the cluster** as indicated in this [serverfault.com discussion](https://serverfault.com/questions/978644/deploy-minio-distributed-on-3-nodes-with-1-drive).

Two solutions are thus possible:

- Either add more disks per node (2 disks per node * 3 nodes = 6, OK)
- Or add one more worker node

Because we could afford the disk space and the RAM, we decided to apply both solutions: MinIO recommends 4 nodes for high-availability, and more volumes helps MinIO be more efficient as well.

### MinIO tenant pods are created, but are pending/crashing

We encountered multiple issues while trying to deploy the MinIO tenants using the MinIO Operator.

At first, we used Helm in order to deploy both the MinIO Operator and MinIO tenant. However, when deploying the tenant:

- All pods were in a pending state;
- PVCs existed, but no PV were provided through the `directpv-min-io` StorageClass provided by DirectPV.

We have tried to modify the tenant deployment without finding a fix.

In order to debug this, we have used another approach to deploy MinIO Operator and tenant altogether, with Krew:

```
kubectl krew install minio
kubectl minio init
kubectl minio tenant create miniotenant --capacity 30Gi --servers 4 --volumes 8 --namespace minio --storage-class directpv-min-io
```

With this approach, we have obtained a different result: this time, the MinIO tenant pods were crashing instead of pending. PVs were provided by DirectPV.

![Image describing the cluster state as described above](./img/issue-minio-krew.png)

We have thus tried to investigate what happened in those crashing pods. A question thus arised: **how do we check the logs for what happened during this crash in Kubernetes?**

Here are useful commands to debug the cluster that we have used:

```
kubectl get events -n <namespace>
kubectl describe pod -n <namespace> <name-of-pod>
```

In this case, the containers themselves were crashing and no event were giving any detail. What we instead did was SSH to one of the nodes where a pod was deployed and checked the container log inside `/var/log/containers/`. For the above error, we have found the following error:

```
{"log":"ERROR Unable to validate passed arguments in MINIO_ARGS:env+tls://j1tu3D5iKzQ8JdzBEalh:gc0ESQtVgTj9QYIyFmCnl4kV4PvKzNPG4sQFxVlW@operator.minio-operator.svc.cluster.local:4222/webhook/v1/getenv/minio/miniotenant: Get \"https://operator.minio-operator.svc.cluster.local:4222/webhook/v1/getenv/minio/miniotenant?key=MINIO_ARGS\": dial tcp 82.66.189.11:4222: i/o timeout\n","stream":"stdout","time":"2022-12-30T14:16:54.131896526Z"}
```

What is odd is that the container tried to reach `82.66.189.11:4222`, which corresponds to none of the services found when using `kubectl get service -A` (in the below screenshot, Trino was not deployed):

![List of services in the cluster](./img/issue-services.png)

## Links used to fix issues

- [Fixing kubeadm not starting because of kubelet](https://stackoverflow.com/questions/57648829/how-to-fix-timeout-at-waiting-for-the-kubelet-to-boot-up-the-control-plane-as-st)
- [Resizing disks on Linux (for our host VM)](https://www.msp360.com/resources/blog/linux-resize-partition/)
- [Debugging Kubelet service not starting on machines](https://serverfault.com/questions/877136/debug-kubelet-not-starting)