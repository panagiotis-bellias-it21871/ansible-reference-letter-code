# Ansible-Reference-Letter-Code
Ansible-Reference-Letter-Code, an ansible project in the context of 'Usage of DevOps Methodologies and Tools in progressive web app development' thesis of HUA-DIT

<p align="left"> <img src="https://komarev.com/ghpvc/?username=pan-bellias&label=Profile%20views&color=0e75b6&style=flat" alt="pan-bellias" /> </p>

<h3 align="left">Languages and Tools:</h3>
<p align="left"> <a href="https://azure.microsoft.com/en-in/" target="_blank"> <img src="https://www.vectorlogo.zone/logos/microsoft_azure/microsoft_azure-icon.svg" alt="azure" width="40" height="40"/> </a> <a href="https://www.gnu.org/software/bash/" target="_blank"> <img src="https://www.vectorlogo.zone/logos/gnu_bash/gnu_bash-icon.svg" alt="bash" width="40" height="40"/> </a>
<a href="https://www.docker.com/" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-original-wordmark.svg" alt="docker" width="40" height="40"/> </a>
<a href="https://git-scm.com/" target="_blank"> <img src="https://www.vectorlogo.zone/logos/git-scm/git-scm-icon.svg" alt="git" width="40" height="40"/> </a> <a href="https://www.jenkins.io" target="_blank"> <img src="https://www.vectorlogo.zone/logos/jenkins/jenkins-icon.svg" alt="jenkins" width="40" height="40"/> </a> <a href="https://kubernetes.io" target="_blank"> <img src="https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg" alt="kubernetes" width="40" height="40"/> </a> <a href="https://www.linux.org/" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/linux/linux-original.svg" alt="linux" width="40" height="40"/> </a> <a href="https://www.nginx.com" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/nginx/nginx-original.svg" alt="nginx" width="40" height="40"/> </a> <a href="https://www.postgresql.org" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/postgresql/postgresql-original-wordmark.svg" alt="postgresql" width="40" height="40"/> </a> <a href="https://www.python.org" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python" width="40" height="40"/> </a> </p>

<a name="contents"></a>
## Table Of Contents
1. [Table Of Contents](#contents)  
2. [Ansible Installation](#installation)  
3. [Connectivity](#conn)  
4. [Deployment Support](#deployment)  
4.1. [Docker](#docker)  
4.1.1. [Containers For FastAPI App](#fastapi)
4.1.2. [Containers For VueJS App](#vuejs)
4.2. [Kubernetes Deployment Usage](#k8s)  
5. [SSL Configuration using playbooks](#ssl)  
6. [files folder](#files)  

<a name="installation"></a>
## Ansible Installation
* [Instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu)

<a name="connect"></a>
## Connectivity

* Create an inventory file (e.g. hosts.yml) that holds the remote hosts that ansible will handle. The file will have entries look like:
```nano
---
# that is a group entry
<group-name>:
    hosts:
        # this is a host entry
        <vm-name>:
            ansible_host: <DNS Name of VM or public IP>
            ansible_port: 22 # because 22 port is for SSH Connection
            ansible_ssh_user: <username in VM>
# in case you have already ssh connection in the VM using ~/.ssh/config no more info are needed like:
<group-name>:
    hosts:
        <vm-name> # The VM name must be the same as the HostName in the ~/.ssh/config
```
For testing you can use either [vagrant](https://www.vagrantup.com/) using [VirtualBox](https://www.virtualbox.org/) making locally VMs or create some extra VMs in some cloud provider like Azure or Okeanos.

* Run
```bash
ansible -m ping all
```
to check connectivity with declared hosts

<a name="deployment"></a>
## Deployment Support

<a name="docker"></a>
### Docker
* Make sure you have configured the inventory file and you are now in the root folder of this project.
<a name="fastapi"></a>
#### Containers For FastAPI App

* Postgres Installation

[postgres-install.yml](playbooks/postgres-install.yml): This playbook installs all the packages needed, starts the postgresql service and creates a database and a user for our fastapi and vuejs project according to values passed during execution from the command line. Also we declare explicitly to which group of hosts we want to install our service. 
```bash
ansible-playbook -l <group-name> playbooks/postgres-install.yml \
-e PSQL_USER=<username-for-db-user> \
-e PSQL_PASSWD=<password-for-db-user> \
-e PSQL_DB=<name-for-our-database>
```

e.g.
```bash
ansible-playbook -l test playbooks/postgres-install.yml \
> -e PSQL_USER=fastapi \
> -e PSQL_PASSWD=pass123 \
> -e PSQL_DB=ref_letters_db
```

Here, we do twice this action with different parameters in a way that both fastapi project and keycloak service are satisfied.

e.g.
```bash
ansible-playbook -l test playbooks/postgres-install.yml \
> -e PSQL_USER=keycloak \
> -e PSQL_PASSWD=pass123 \
> -e PSQL_DB=auth_db
```

* Private Repository (configure like this in a way Ansible will be able to clone the private repo in the remote server)
[source](https://ntlx.org/de/2018/07/use-ansible-to-clone-update-private-git-repositories-via-ssh.html)
```bash
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa # add your private ssh key you use to connect to github to ssh-agent
```

[keycloak-install.yml](playbooks/keycloak-install.yml)


[fastapi-install.yml](playbooks/fastapi-install.yml): This playbook installs docker and docker-compose packages, clones fastapi project code, populate .env variables, and scales up the containers declared in the docker-compose.yml of fastapi project taking build instructions from fastapi.nonroot.Dockerfile of our app, according also to values passed during execution from the command line. Also we declare explicitly to which group of hosts we want to deploy our app.
```bash
ansible-playbook -l <group-name> playbooks/fastapi-install.yml \
-e DATABASE_URL=<url-where-database-runs-with-right-credentials> \ # e.g. postgresql://demouser:pass123@db/deploy_db
```
The whole operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile.

e.g.
```bash
ansible-playbook -l test playbooks/fastapi-install.yml \
> -e DATABASE_URL=postgresql://localhost/ref_letters_db?user=fastapi&password=pass123
```
<a name="vuejs"></a>
#### Containers For VueJS App

[vuejs-install.yml](playbooks/vuejs-install.yml): This playbook installs docker and docker-compose packages, clones vuejs project code, populate .env variables, and scales up the containers declared in the docker-compose.yaml of vuejs project taking build instructions from Dockerfile of our app, according also to values passed during execution from the command line. Also we declare explicitly to which group of hosts we want to deploy our app.
```bash
ansible-playbook -l <group-name> docker/fastapi-install.yml \
-e VUE_APP_BACKEND_URL="http://127.0.0.1:8000" \
-e VUE_APP_KEYCLOAK_URL="http://127.0.0.1:8001/user"
```
This operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile.

<a name="k8s"></a>
### Kubernetes Deployment Usage
* The [populate-env.yml](playbooks/populate-env.yml) is used to populate the .env variables after we have cloned locally our projects and before we will be applying the k8s .yaml files to have entities creation. Because a configmap must be generated from .env file we have locally, so the playbook doesn't handle some remote host but only localhost (No need to declare group of hosts). These values are passed from the command line like before.
```bash
ansible-playbook playbooks/populate-env.yml \
-e DATABASE_URL=<url-where-database-runs-with-right-credentials> \ # e.g. postgresql://testuser:pass1234@pg-cluster-ip/demo_db
-e VUE_APP_BACKEND_URL="http://127.0.0.1:8000" \
-e VUE_APP_KEYCLOAK_URL="http://127.0.0.1:8001/user"
```
This operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile with the rest commands needed to deploy on a k8s cluster.

<a name="ssl"></a>
## SSL Configuration using playbooks

* [docker-https](playbooks/docker-https.yml): This is used only manually so as to configure SSL certificates for HTTPS environment in docker-vm.
* [jenkins-config](playbooks/jenkins-config.yml): This is used so as to configure SSL certificates for HTTPS environment in jenkins-server for extra security.

<a name="files"></a>
## files folder

* [fastapi](files/fastapi): has a file that describes how the uvicorn service is built and running in a VM.
* [nginx](files/nginx): has configuration files for nginx sites like the vuejs app and the jenkins service, both in HTTP and HTTPS.

For HTTPS you should make a folder named 'certs' under [files folder](files) and there you have to store (and concatenate according ZeroSSL instructions) your SSL Certificates for your ansible-vm under websystem subfolder, and your jenkins-server under jenkins subfolder.

## It's our pleasure to contact us at our social media or at github [issues](https://github.com/pan-bellias/Ansible-Reference-Letter-Code/issues)