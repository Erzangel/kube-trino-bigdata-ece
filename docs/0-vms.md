# Setting up VMs for Kubernetes

Our cluster's creation is going to be automated. This goes from the **creating of the hosts of the cluster themselves**.

In order to achieve this, two mains tools exist to create & subsequently provision VMs automatically:

- Vagrant (TODO: Insert link)
- Terraform

## Vagrant

Vagrant is a tool developed by Hashicorp. It is a popular and powerful Infrastructure as Code (IaC) tool.

Vagrant is able to create VMs using different virtualization providers. Some providers include:

- VirtualBox
- VMware
- KVM & QEMU

Ideally, KVM & QEMU are preferred for their speed & proximity to the original machine's kernel.

We have tried to use libvirt, but because the host we are using for this projet does not support KVM, we use VirtualBox instead. Our steps for using libvirt are still documented.


### Installing VirtualBox provider

```
sudo apt get update
sudo apt install virtualbox
```

### Installing libvirt

For `libvirt` to be used on a host, we must first verify that the KVM module is loaded and that our system supports it with `kvm-ok`.

We install all libvirt utilities on our Ubuntu host:

```
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
```

We also need to install the vagrant-libvirt plugin for libvirt boxes support:

```
vagrant plugin install vagrant-libvirt
```

## Vagrantfile

```
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu1804"
  config.vm.provider :libvirt do |libvirt|
    libvirt.graphics_type = "none"
	libvirt.memory = 3072
    libvirt.cpus = 4
    libvirt.storage :file, :size => '5G', :type => 'qcow2'
  end
end
```