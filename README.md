# Trino deployment on Kubernetes - ECE Paris Big Data Course Project

## About

This repository offers everything required to deploy a distributed data processing environment on Kubernetes over 4 virtual machines.

It has been developed in the context of a project in the Big Data Ecosystem course of ECE Students Luka, Cléa & Mathias.

It aims to **deploy Trino in Kubernetes**, while relying on **object storage with MinIO**.

## Usage

1. Adjust the machine's resources in the `Vagrantfile` to match your own host computer's limitations. (Bear in mind that 6 GB per worker should be seen as a minimum, and that MinIO requires at least 4 hard disks to function)
2. Clone this repository
3. Install Vagrant, VirtualBox, Ansible on your host machine
4. Run `vagrant up` in the root of the repository

## Architecture we used

We used a Virtual Machine hosted in a friend's server, running Ubuntu 22.04. The specs are as follows:

- CPU: 16 cores
- RAM: 94 GB
- Storage: 256 GB

We have previously attempted to run this project in a Linux Container (LXC) with the same specs, but the latter supported neither VirtualBox nor KVM/QEMU. We thus switched to a traditional VM, which supports VirtualBox.

## Documentation index

We have documented all of our attempts for each part of the project, which we have divided and documented as follows:

0. [Creating & provisioning VMs with Vagrant](./docs/0-vms.md)
1. [Deploying Kubernetes with kubeadm, kubelet, kubectl](./docs/1-deploying-kubernetes.md)
2. [Deploying basic Kubernetes services (CNI)](./docs/2-basic-kubernetes-services.md)
3. [Deploying MinIO to provide object storage](./docs/3-minio-object-storage.md)
4. [Deploying Trino to perform distributed computation](./docs/4-trino-processing.md)

## Contributors

| Name              | Email                       |
| ---------------- | --------------------------- |
| Luka BIGOT       | luka.bigot@edu.ece.fr       |
| Cléa DEDUIT      | clea.deduit@edu.ece.fr      |
| Mathias SERICOLA | mathias.sericola@edu.ece.fr |
