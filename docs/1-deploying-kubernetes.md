# Deploying Kubernetes

In order to deploy Kubernetes on our machine, we have decided to use `kubeadm`.

We have followed the official guidelines provided [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/), which indicate the following steps:

- Installing a container runtime
- Installing `kubeadm`, `kubelet` and `kubectl`
- Running the commands to create the cluster

## Installing a container runtime: Docker

We have followed the official documentation to install Docker on our Ubuntu VMs, which can be found [here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).

To sum it up, we need to install the following packages as prerequisites: `ca-certificates`, `curl`, `gnupg` and `lsb-release`.

Then, we run the following commands:

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

As we use an IaC approach in our project, we are going to automate all of those steps as tasks in an Ansible role. Said role can be found [here](). TODO: Insert link

Note: The commands described above can be run with Ansible using the `command` or `shell` module. However, as described in [this article](https://www.ansiblepilot.com/articles/ansible-modules-command-vs-shell/), it is preferred to use built-in Ansible modules when possible.

Because we add an apt key and an apt repository, we use the corresponding modules in Ansible:

```
- name: Adding the apt signing key for Docker repository
  become: yes
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Adding apt repository for stable version of Docker
  become: yes
  apt_repository:
    repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable
    state: present
```

In the same way, when we install packages, we use the `apt` module:

```
- name: Installing prerequisites packages
  become: yes
  apt:
    name: "{{ packages }}"
    state: present
    update_cache: yes
  vars:
    packages:
    - ca-certificates
    - curl
    - gnupg
    - lsb-release
    - apt-transport-https
```

## Installing kubeadm, kubelet, kubectl

The official documentation provides steps to install each of the components [in this section of the above link](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl).

In a nutshell, it requires installing the `apt-transport-https`, `ca-certificates` and `curl` packages and running the following commands:

```
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Once again, we run those commands in the same Ansible role referenced above, using the dedicated Ansible modules.