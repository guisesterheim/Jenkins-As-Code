# -*- mode: ruby -*-
# vi set ft=ruby :

Vagrant.configure(2) do |config|

    config.vm.provider "virtualbox"
    config.vm.provider "virtualbox" do |v|
        v.memory = 4096
        v.cpus = 4
    end

    config.vm.box = "ubuntu/xenial64"
    config.vm.network "forwarded_port", guest: 8080, host: 5555, host_ip: "127.0.0.1"
    config.vm.network "forwarded_port", guest: 80, host: 6666, host_ip: "127.0.0.1"
    config.vm.synced_folder ".", "/home/vagrant/shared"
    config.vm.provision "shell", inline: <<-SHELL

        sudo apt-get update -y
        sudo apt-get install -y wget curl vim git unzip

        # Install PIP
        sudo apt-get install -y python3
        sudo apt-get install -y python3-pip
        sudo pip3 install --upgrade pip
        sudo pip3 install packaging
        
        # Install Azure CLI
        curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

        # Install Ansible
        sudo apt-get -y install python-jinja2 python-paramiko
        sh -c 'mkdir -p /home/ubuntu/'
        sh -c 'touch /home/ubuntu/ansible_hosts'
        sh -c 'echo "[localhost]" > /home/ubuntu/ansible_hosts'
        sh -c 'echo "localhost ansible_connection=local" >> /home/ubuntu/ansible_hosts'
        sh -c 'echo "export ANSIBLE_INVENTORY=~/ansible_hosts" >> /etc/profile'
        sudo pip3 install ansible

        # Clone repo and start the app
        git clone https://github.com/guisesterheim/Jenkins-As-Code.git /home/ubuntu/jenkins

        # Preset Jenkins config files
        sudo mkdir -p /var/lib/jenkins
        sudo cp /home/ubuntu/jenkins/jenkins.yaml /var/lib/jenkins

        # Run playbook
        sudo ansible-playbook /home/ubuntu/jenkins/ansible_config/site.yml
        
        # Change users permissions for docker
        sudo usermod -aG docker jenkins
        sudo usermod -aG root jenkins
        sudo chmod 777 /var/run/docker.sock

        # Wait for jenkins to restart
        sleep 60s

        # Check if the service is up, running and responding
        sudo bash /home/ubuntu/jenkins/ansible_config/general-test.sh
    SHELL
end