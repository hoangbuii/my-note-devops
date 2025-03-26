I'm a Devops Engineer
=====================
> This note is best for Ubuntu. If you have another DISTRO, looking for guide in the internet, it's simmilar to my note 
## 1. install tool
> First of all, we have to install tools that necessary for your project
### 1.1 Docker
* Install Docker
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

* Manage Docker as a non-root user

    * Create the docker group.
    ```bash
    sudo groupadd docker
    ```
    * Add your user to the docker group
    ```bash
    sudo usermod -aG docker $USER
    ```
    * Log out and log back in so that your group membership is re-evaluated.
    ```bash
    newgrp docker
    ```
    * Verify that you can run docker commands without sudo.
    ```bash
    docker ps
    ```
### 1.2 Ansible
* Installing Ansible
```bash
sudo apt update
sudo apt install ansible -y
```
* Write inventory file (can use default name is `inventory.ini`)
```bash
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com

```
* Useful Ansible Commands
    * Check syntax:
    ```bash
    ansible-playbook --syntax-check playbook.yml
    ```
    * Run in dry-run mode (no changes made):

    ```bash
    ansible-playbook -i inventory.ini playbook.yml --check
    ```
    * Check hosts for connection:

    ```bash
    ansible -i inventory.ini all -m ping
    ```

### 1.3 Kubernetes
#### 1.3.1 Kubernetes in VM
1. Requirements
    * VM that responsible as master node and worker node

    | VM | Minimum Memory | OS | Role | Require Pakage |
    | --- | --- | --- | --- | --- |
    | VM 1 | 1500MB | Ubuntu | Master Node | None |
    | VM 2 | 1024MB | Ubuntu | Worker Node | None | 
    | ... | ... | ... | ... | ... |
    | VM n | 1024MB | Ubuntu | kubespray | Docker, Ansible, Python |

    * [Ansible](#12-ansible)
    * [kubespray](https://github.com/kubernetes-sigs/kubespray)
    * [Docker](#11-docker)
    * Python
2. Install 
    * Get kubespray repository
    ```bash
    git clone https://github.com/kubernetes-sigs/kubespray
    cd kubespray
    ```
    * Install and active Python venv
        * Install `python-venv`
        ```bash
        sudo apt install python3-venv
        ```
        * Active to python venv
        ```bash
        python3 -m venv .kube-venv
        source .kube-venv/bin/activate
        ```
        * Install necessary package
        ```bash
        pip install -r requirements.txt
        pip install -r contrib/inventory_builder/requirements.txt
        ```
        * Ensure kubespray VM can ssh to all node without password (see in [Generate SSH key](#set-up-ssh-using-ssh-key) )
    * Prepare host file
        * Copy `inventory/sample` as `inventory/mycluster`
        ```bash
        cp -rfp inventory/sample inventory/mycluster
        ```
        * Update Ansible inventory file with inventory builder (change IP to actual IP address of VM)
        ```bash
        declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
        CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
        ```
        * note: you must add `ansible_user: root` to list host
        * Review and change parameters under `inventory/mycluster/group_vars`
        ```bash
        cat inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml
        ```
        May modify Container Runtime at `container_manager:`, and Network Plugin at `kube_network_plugin: `
    * Run Kubespray container
    ```
    git checkout v2.26.0
    docker pull quay.io/kubespray/kubespray:v2.26.0
    docker run --rm -it \
        --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa \
        --mount type=bind,source="$(pwd)"/inventory/mycluster,dst=/inventory \
        quay.io/kubespray/kubespray:v2.26.0 bash
    ```
    * Run playbook to build k8s cluster
    ```
    ansible-playbook -i /inventory/hosts.yaml  --become --become-user=root cluster.yml
    ```
    * Remove all installation
    ```
    ansible-playbook -i /inventory/hosts.yaml  --become --become-user=root reset.yml
    ```
#### 1.3.2 install minikube in ubuntu
1. Install Docker
* Set up Docker's apt repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
* Install the Docker packages.
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y 
```
* Ensure Docker be manageable as a non-root user
```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker ps
```
2. Install minikube
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```
3. Install kubectl 
```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
4. Start Minikube with Docker
```
minikube start --driver=docker
```
5. Verify installation
* Check status of MiniKube installation
```
minikube status
```
* Check kubectl status
```
minikube kubectl -- get node -owide
```
6. (Optional) Reduce minikube command
```
alias kubectl="minikube kubectl --"
```

### 1.4 kubectl
* Install kubectl 
```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.30.0/2024-05-12/bin/linux/amd64/kubectl
chmod +x kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
mkdir -p ~/.kube
```
* Defalut configuration file of k8s is in `/etc/kubernetes/admin.conf`, We need to copy it to defalut `$KUBECONFIG`
```
sudo chown $(id -u):$(id -g) ~/.kube/config
export KUBECONFIG=~/.kube/config
export PATH="kubectl:$PATH"
source <(kubectl completion bash)
```

### 1.5 Helm
```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
# Some tools
## Set up ssh using SSH-key
1. Generate SSH Key Pair on the Client `-C` is **optional**
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
* When prompted:

    * Enter a file in which to save the key (or press **Enter** to accept the default location, typically `~/.ssh/id_rsa`).
    * Enter a passphrase (optional but recommended for added security).
2. Copy the Public Key to the Server
* Show the key from client
```bash
cat ~/.ssh/id_rsa.pub
```
* Copy this key and append to `~/.ssh/authorized_keys` from server seprate by `endline`
```bash
cat ~/.ssh/authorized_keys
```
* **note**: if you have private key in some where and copy to `~/.ssh/id_rsa`, the `~/.ssh/id_rsa.pub` must be removed and you have to change permission for this private key:
```bash
chmod 600 ~/.ssh/id_rsa
```
3. Verify SSH Key-Based Authentication
* If private key is `~/.ssh/id_rsa`
```bash
ssh username@server_ip
```
* sepeci key
```bash
ssh -i ~/.ssh/new_key_name username@server_ip
```




















> patten
```bash

```
