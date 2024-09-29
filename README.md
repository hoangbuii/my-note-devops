I'm a Devops Engineer
=====================

## 1. install tool

### 1.1 Docker
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