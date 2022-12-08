# Trino deployment on Kubernetes - ECE Paris Big Data Course Project

## About

This repository holds the Big Data Ecosystem project of ECE Students Luka, Cléa & Mathias.

It aims to **deploy Trino in Kubernetes**, while relying on **object storage with MinIO**.

## Usage

`TO-DO`

## Architecture we used

We used a Virtual Machine hosted in a friend's server, running Ubuntu 22.04. The specs are as follows:

- CPU: 16 cores
- RAM: 24 GB
- Storage: 30 GB

## Documentation index

We have documented all of our attempts for each part of the project, which we have divided and documented as follows:

- [0. Creating & provisioning VMs with Vagrant](./docs/0-vms.md)
- 1. Deploying Kubernetes with kubeadm, kubelet, kubectl
- 2. Deploying basic Kubernetes services (load balancer, Ingress)
- 3. Deploying MinIO to provide object storage
- 4. Deploying Trino to perform distributed computation

## Contributors

| Name              | Email                       |
| ---------------- | --------------------------- |
| Luka BIGOT       | luka.bigot@edu.ece.fr       |
| Cléa DEDUIT      | clea.deduit@edu.ece.fr      |
| Mathias SERICOLA | mathias.sericola@edu.ece.fr |
