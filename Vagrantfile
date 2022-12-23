# Vagrantfile for the kube-trino-bigdata-ece repository
# This file was inspired by the one already developed by Luka Bigot here:
# https://github.com/adaltas/demo-k8s-mini/blob/main/Vagrantfile

# ======================= CONTEXT =========================

# The goal here is to deploy 4 machines in total:
# - 1 master node with a bit less resources, to run the Kubernetes control plane
# - 3 worker nodes with both compute resources and storage attached

# As explained in the docs, we decided to use VirtualBox as a virtualization backend,
# but it is very much possible to modify this Vagrantfile to use libvirt instead
#
# Then, Ansible is run on all machines in parallel:
# each task runs on each machine before proceeding to the next task.
# This is particularly useful in our Kubernetes deployment context

# The host machine we used has the following specs:
# - OS: Ubuntu 20.04
# - RAM: 24 GB
# - Free storage: 15 GB

# ====================== PARAMETERS =======================

# Memory in MB and amount of CPU cores 
# for the master and worker nodes
MASTER_MEMORY = 8192
MASTER_CPUS = 4
WORKER_MEMORY = 16384
WORKER_CPUS = 4

# Number of disks per worker node
# and size of an individual disk in GB
WORKER_DISKS_NB = 2
WORKER_DISK_SIZE = 10

# VM image "box" name used by Vagrant
VM_BOX_NAME = "bento/ubuntu-20.04"

# =========================================================

# Define the array of machines using the above parameters
cluster = {
  "kube-trino-master" => { :ip => "192.168.58.10", :memory => MASTER_MEMORY, :cpus => MASTER_CPUS, :disks => 0 },
  "kube-trino-worker-01" => { :ip => "192.168.58.11", :memory => WORKER_MEMORY, :cpus => WORKER_CPUS, :disks => WORKER_DISKS_NB },
  "kube-trino-worker-02" => { :ip => "192.168.58.12", :memory => WORKER_MEMORY, :cpus => WORKER_CPUS, :disks => WORKER_DISKS_NB },
  "kube-trino-worker-03" => { :ip => "192.168.58.13", :memory => WORKER_MEMORY, :cpus => WORKER_CPUS, :disks => WORKER_DISKS_NB },
  "kube-trino-worker-04" => { :ip => "192.168.58.14", :memory => WORKER_MEMORY, :cpus => WORKER_CPUS, :disks => WORKER_DISKS_NB },
}


Vagrant.configure("2") do |config|
  # Iterate over each entry in the dictionary declared above,
  # with its key renamed as host_name and its values as host_specs
  cluster.each_with_index do |(host_name, host_specs), index|
    config.ssh.insert_key = false
    config.vm.define host_name do |node|
      node.vm.hostname = host_name
      node.vm.box = VM_BOX_NAME
      node.vm.provider :virtualbox do |vb|
        # DNS settings to minimize interferences with the kube DNS
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
	vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
        # Memory and CPU allocation
        vb.memory = host_specs[:memory]
	vb.cpus = host_specs[:cpus]
        # Disk setting in VirtualBox, if there is a disk number indicated in specs
	(0...host_specs[:disks]).each do |i|
	  vb.customize ["modifyvm", :id, "--ioapic", "on"]
          # Creating a virtual disk called "<VM-name>-disk-n" with a size of 30GB
          vb.customize ["createhd", "--filename", "#{host_name}-disk-#{i}", "--size", WORKER_DISK_SIZE * 1024]
          # Attaching the newly created virtual disk to our node
          vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 3 + i, "--device", 0, "--type", "hdd", "--medium", "#{host_name}-disk-#{i}.vdi"]
	end
      end
      # Network config
      node.vm.network "private_network", ip: "#{host_specs[:ip]}"
      # Once we create the last machine, run the Ansible provisioner
      # and provide arguments to, in fact, run Ansible on all VMs
      if index == cluster.length() - 1
        node.vm.provision :ansible do |ansible|
	  ansible.limit = "all"
	  ansible.playbook = "kube-trino-playbook.yml"
	  # Possible to fork the Ansible process more
	  # in order to speed up provisioning
	  ansible.raw_arguments = ["--forks=#{cluster.length()}"]
	end
      end
    end
  end
end
