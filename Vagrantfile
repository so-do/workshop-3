# -*- mode: ruby -*-
# vi: set ft=ruby :

KUBERNETES_VERSION = ENV['KUBERNETES_VERSION'] || "1.11.4"

$docker = <<SCRIPT
#!/bin/bash

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get -y update
sudo apt-get install -y docker-ce=17.03.3~ce-0~ubuntu-$(lsb_release -cs)
sudo systemctl start docker
sudo usermod -a -G docker vagrant
SCRIPT


# Common installation script
$installer = <<SCRIPT
#!/bin/bash

# Update apt and get dependencies
sudo apt-get -y update
sudo apt-get install -y zip unzip curl wget socat ebtables git vim tmux mc htop

docker pull redis

git clone https://github.com/so-do/workshop-3.git
git clone https://github.com/kubernetes-sigs/kubespray.git
SCRIPT


$minikubescript = <<SCRIPT
#!/bin/bash

#Install minikube
echo "Downloading Minikube"
curl -q -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 2>/dev/null
chmod +x minikube 
sudo mv minikube /usr/local/bin/

#Install kubectl
echo "Downloading Kubectl"
curl -q -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/v${KUBERNETES_VERSION}/bin/linux/amd64/kubectl 2>/dev/null
chmod +x kubectl 
sudo mv kubectl /usr/local/bin/

# Install crictl
curl -qL https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.12.0/crictl-v1.12.0-linux-amd64.tar.gz 2>/dev/null | tar xzvf -
chmod +x crictl
sudo mv crictl /usr/local/bin/

# Install helm
curl -qL https://get.helm.sh/helm-v2.14.2-linux-amd64.tar.gz 2>/dev/null | tar xzvf -
sudo mv linux-amd64/helm /usr/local/bin/helm
sudo mv linux-amd64/tiller /usr/local/bin/tiller
rm -rf linux-amd64

#Setup minikube
echo "127.0.0.1 minikube minikube." | sudo tee -a /etc/hosts
mkdir -p $HOME/.minikube
mkdir -p $HOME/.kube
touch $HOME/.kube/config

export KUBECONFIG=$HOME/.kube/config

# Permissions
sudo chown -R $USER:$USER $HOME/.kube
sudo chown -R $USER:$USER $HOME/.minikube

export MINIKUBE_WANTUPDATENOTIFICATION=false
export MINIKUBE_WANTREPORTERRORPROMPT=false
export MINIKUBE_HOME=$HOME
export CHANGE_MINIKUBE_NONE_USER=true
export KUBECONFIG=$HOME/.kube/config

# Disable SWAP since is not supported on a kubernetes cluster
sudo swapoff -a

## Start minikube 
sudo -E minikube start -v 4 --vm-driver none --kubernetes-version v${KUBERNETES_VERSION} --bootstrapper kubeadm 

## Addons 
sudo -E minikube addons  enable ingress

## Configure vagrant clients dir

printf "export MINIKUBE_WANTUPDATENOTIFICATION=false\n" >> /home/vagrant/.bashrc
printf "export MINIKUBE_WANTREPORTERRORPROMPT=false\n" >> /home/vagrant/.bashrc
printf "export MINIKUBE_HOME=/home/vagrant\n" >> /home/vagrant/.bashrc
printf "export CHANGE_MINIKUBE_NONE_USER=true\n" >> /home/vagrant/.bashrc
printf "export KUBECONFIG=/home/vagrant/.kube/config\n" >> /home/vagrant/.bashrc
printf "source <(kubectl completion bash)\n" >> /home/vagrant/.bashrc

# Permissions
sudo chown -R $USER:$USER $HOME/.kube
sudo chown -R $USER:$USER $HOME/.minikube

# Enforce sysctl 
sudo sysctl -w vm.max_map_count=262144
sudo echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.d/90-vm_max_map_count.conf

SCRIPT

def configureVM(vmCfg, hostname)
  vmCfg.vm.hostname = hostname

#  vmCfg.vm.network "private_network", type: "dhcp",  :model_type => "virtio", :autostart => true

  # ensure docker is installed # Use our script so we can get a proper support version
  vmCfg.vm.provision "shell", inline: $docker, privileged: false 
  # Script to prepare the VM
  vmCfg.vm.provision "shell", inline: $installer, privileged: false 
  vmCfg.vm.provision "shell", inline: $minikubescript, privileged: false, env: {"KUBERNETES_VERSION" => KUBERNETES_VERSION}

  return vmCfg
end


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/xenial64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 443, host: 8443, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 30000, host: 30000, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL

   hostname = 'minikube-vagrant'

   config.vm.define hostname do |vmCfg|
     vmCfg = configureVM(vmCfg, hostname)  
   end
end
