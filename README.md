# Jenkins Config

If you need to...
- Build a new AMI image, go for Packer section
- Test a new image build locally (before deploying to Azure), go for Vagrant section
- Change Jenkins configuration before testing locally, go for Jenkins configuration section
- Change Jenkins pipeline or credentials, go for Jenkins Pipeline section
- Change how to OS behaves, go for Ansible section 
- Deploy a new instance of Jenkins Master >> Go for Deploying a new Jenkins

### Jenkins configuration files

The ansible roles listed below will configure Jenkins. The plugins this Jenkins uses can be found at: <code>ansible_config/roles/ansible-role-jenkins/defaults/main.yml</code>. This jenkins is configured automatically using Jenkins plugin <code>configuration as code</code>. All the configuration is listed on file <code>jenkins.yaml</code> in this root

### Jenkins Pipeline and credentials files

File jenkins.yaml describes all the pipelines this Jenkins will have. Each pipeline points to a Jenkinsfile located whether in <code>jenkins_infra</code> or in each service folder as "JenkinsFile".

## Other tools - how this repository works

### Packer

Packer is a tool to create an image (VM on Azure OR AMI on AWS)

Running packer:
1. <code>packer build -var 'client_id=<client_id>' -var 'client_secret=<client_secret>' -var 'subscription_id=<subscription_id>' -var 'tenant_id=<tenant_id>' packer_config.json</code>

Checkout the file <code>packer_config.json</code> to see how packer will create your SO image and Azure instructions for it


### Ansible

Ansible is a tool to configure our OS as we want it to be.

You can run ansible with: <code>ansible playbook site.yml</code>. See examples at <code>Vagrantfile</code> and <code>packer_config.json</code>

The main file for this folder is <code>ansible_config/site.yml</code>. This file calls all the roles in "roles" folder

#### Ansible roles:

The roles folder has the Ansible configuration for:
1. Add Java PPA
2. Role - Install Java JDK 8
3. Role - Install Liquibase
4. Role - Install Docker
5. Role - Install Terraform
6. Role - Install Kubectl
7. Role - Install Jenkins (with plugins and pipelines configuration)
8. Role - Install HAProxy to handle the server TLS

### Vagrant - build your Jenkins image locally

Vagrantfile is used to local tests only. This is a pre-step before creating the image on Azure with Packer

#### Vagrant commands:

- Have vagrant installed (like sudo apt install vagrant) and Oracle's VirtualBox

- How to run: navigate to root of this repo and run <code>sudo vagrant up</code>. After everything is complete, it will create a Jenkins acessible from your host machine at <code>localhost:5555</code> and <code>localhost:6666</code>

This will create a virtual machine and will install everything listed on the Vagrantfile

- How to SSH into the created machine: run <code>sudo vagrant ssh</code>

- How to destroy the VM: run <code>sudo vagrant destroy</code>


## Redeploying Jenkins - activating TLS (https) and SSO afterwards

Once you need to change configurations and deploy a new Jenkins instance, you should:

1. Test your configurations manually. Once everything is set
2. Apply your configurations the same way you did manually but now using Ansible and its roles
3. Test it locally using Vagrant. Once everything is set
4. Use packer to build your new image at Azure

### Once you have your machine up and running, connect through SSH to perform the last manual steps: TLS and SSO Google authentication:

1. Generate the .pem certificate file with <code>cat STAR.mycompany.com.crt STAR.mycompany.com.key > fullkey.pem</code>. Infra has the certificate files should you need it (remember to remove the empty row that is kept inside the generated fullkey.pem between the two certificates. To look at the file use <code>cat fullkey.pem</code>)
2. Move the generated file to folder <code>/home/ubuntu/jenkins/</code>
3. Restart HAProxy with <code>sudo service haproxy restart</code>
4. Log in to jenkins using regular admin credentials. Go to "Manage Jenkins" > "Global Security". Under "Authentication" select "Login with Google" and paste like below:
> * Client id = client_id
> * Client secret = client_secret
> * Google Apps Domain = mycompany.com