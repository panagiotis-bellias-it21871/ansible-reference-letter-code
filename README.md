# ansible-reference-letter-code
An ansible project about reference letter handling in the context of DIT HUA Thesis "Use of devops methodologies and tools in development and production environment of web applications"

<p align="left"> <img src="https://komarev.com/ghpvc/?username=panagiotis-bellias-it21871&label=Profile%20views&color=0e75b6&style=flat" alt="panagiotis-bellias-it21871" /> </p>

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
<a name="web-system"></a>
#### Containers For Our Web System

[reference-letters-system-install.yml](playbooks/reference-letters-system-install.yml): This playbook installs docker and docker-compose packages, clones fastapi and vuejs projects' code, populate .env variables, and scales up the containers declared in the docker-compose.yml of project taking build instructions from nonroot.Dockerfile of our backend app and from Dockerfile of our frontend app, according also to values passed during execution from the command line. Also we declare explicitly to which group of hosts we want to install and deploy our system.  

** SAMPLE_EXECUTION **

```bash
ansible-playbook -l docker_group playbooks/reference-letters-system-docker-install-demo.yml \
--ask-become-pass -e group='belliaspan' \
-e user_dir='/home/belliaspan' \
-e BACKEND_DIR=reference-letters-fastapi-server -e DATABASE_URL=postgresql://bellias:pass123@postgres_db:5432/reference_letters_data \
-e ORIGINS='["http://localhost:8080/","http://localhost:8081/", "http://vuejs:8080", "http://vuejs:8081"]' \
-e MAIL_USERNAME=ref.letters.web.app@gmail.com \
-e MAIL_PASSWORD=<YOUR-EMAIL-PASSWORD> \
-e MAIL_FROM=ref.letters.web.app@gmail.com -e MAIL_PORT=587 \
-e MAIL_SERVER=smtp.gmail.com -e MAIL_FROM_NAME='Reference Letters Web Application' \
-e SECRET="1237N20^K9*t0HYaBayuo7XwgTg*kXspVXWUIz@ReE7eHxDY" \
-e FRONTEND_DIR=reference-letters-vuejs-client \
-e VUE_APP_BACKEND_URL="http://localhost:8000" \
-e VUE_APP_BASE_ENDPOINT_PREFIX="api" \
-e VUE_APP_RL_LETTERS_ENDPOINT="rl_requests" \
-e VUE_APP_AUTH_ENDPOINT_PREFIX="auth" \
-e VUE_APP_AUTH_LOGIN_ENDPOINT="auth/login" -e HOST=localhost
```
```bash
ansible-playbook -l <group-name> playbooks/reference-letters-system-install.yml \
-e BACKEND_DIR=reference-letters-fastapi-server \
-e DATABASE_URL=<url-where-database-runs-with-right-credentials> \
-e FRONTEND_DIR=reference-letters-vuejs-client \
-e VUE_APP_BACKEND_URL="http://<dns-record-or-public-ip>:8000/api" \
-e HOST=<dns-record-or-public-ip>
```

e.g.
```bash
ansible-playbook -l docker_group playbooks/reference-letters-system-install.yml \
-e BACKEND_DIR=reference-letters-fastapi-server \
-e DATABASE_URL=postgresql://bellias:pass123@postgres_db:5432/reference_letters_data \
-e FRONTEND_DIR=reference-letters-vuejs-client \
-e VUE_APP_BACKEND_URL="http://devops2docker.ddns.net:8000/api" \
-e HOST=devops2docker.ddns.net
```

The whole operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile.


<a name="k8s"></a>
### Kubernetes Deployment Usage
* The [populate-k8s-dotenv.yml](playbooks/populate-k8s-dotenv.yml) is used to populate the .env variables after we have cloned locally our projects and before we will be applying the k8s .yaml files to have entities creation. Because a configmap must be generated from .env file we have locally, so the playbook doesn't handle some remote host but only localhost (No need to declare group of hosts). These values are passed from the command line like before.
```bash
ansible-playbook playbooks/populate-k8s-dotenv.yml  \
> -e PSQL_USER=bellias \
> -e PSQL_PASSWD=pass123 \
> -e PSQL_DB=reference_letters_data \
> -e DATABASE_URL=postgresql://bellias:pass123@pg_cluster_ip/reference_letters_data \
-e VUE_APP_BACKEND_URL="http://fastapi:8000" \
-e VUE_APP_KEYCLOAK_URL="http://keycloak_auth:8085/auth"
```
This operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile with the rest commands needed to deploy on a k8s cluster.

<a name="ssl"></a>
## SSL Configuration using playbooks

* [docker-https](playbooks/docker-https.yml): This is used only manually so as to configure SSL certificates for HTTPS environment in docker-vm.
* [jenkins-config](playbooks/jenkins-config.yml): This is used so as to configure SSL certificates for HTTPS environment in jenkins-server for extra security.

<a name="files"></a>
## files folder

The [files/nginx](files/nginx) folder has configuration files for nginx sites like *the vuejs app and* the jenkins service, both in HTTP and HTTPS.

For HTTPS you should make a folder named 'certs' under [files folder](files) and there you have to store (and concatenate according ZeroSSL instructions) your SSL Certificates for your docker-vm under websystem subfolder, and your jenkins-server under jenkins subfolder.

## It's our pleasure to contact us at our social media or at github [issues](https://github.com/panagiotis-bellias-it21871/ansible-reference-letter-code/issues)*
