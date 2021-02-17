### Install Local Virtual Machine with Vagrant

## Vagrant file content

Vagrant.configure("2") do |config|
  
  config.vm.define "kubeclientazure" do |kubeclient|
    kubeclient.vm.box = "centos/7"
    kubeclient.vm.hostname = "kubeclientazure.example.com"
    kubeclient.vm.network "private_network", ip: "172.42.42.201"
    kubeclient.vm.provider "virtualbox" do |v|
      v.name = "kubeclientazure"
      v.memory = 2048
      v.cpus = 2
      # Prevent VirtualBox from interfering with host audio stack
      v.customize ["modifyvm", :id, "--audio", "none"]
    end
  end
end


## Start the Virtual machine
vagrant up

## Connect to the server
vagrant ssh kubeclientazure


## Install Azure CLI

https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=dnf

### Import the Microsoft repository key.
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc

### Create local azure-cli repository information.
vi /etc/yum.repos.d/azure-cli.repo

echo -e "[azure-cli]
name=Azure CLI
baseurl=https://packages.microsoft.com/yumrepos/azure-cli
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/azure-cli.repo


### install

sudo yum install azure-cli


### Login to azure

az login

### set account
az account set --subscription <subscription_id>

az aks get-credentials --resource-group kubernetes-process-group --name kubernetes-cluster

## Access the cluster

### install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

### change to executable
chmod +x kubectl

### move to /usr/bin

sudo mv kubectl /usr/bin

### access

kubectl get deployments --all-namespaces
