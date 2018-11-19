# Préparation VM sur Digital Ocean

Créer une Droplet CentOS 7.4 ou 7.5

# Install pre-requisite packages

```
sudo yum check-update; sudo yum install -y gcc libffi-devel python-devel openssl-devel epel-release
sudo yum install -y python-pip python-wheel
```

# Install Ansible and Azure SDKs via pip
```
sudo pip install ansible[azure]
```

# Create a resource-group

```
az ad sp create-for-rbac --name ansible-rg
```

Then 
```
{
  "appId": "0cf730b5-9e49-44a8-b163-2626814d1206",
  "displayName": "ansible-rg",
  "name": "http://ansible-rg",
  "password": "1425915e-d539-407d-86e1-c92b68f8724a",
  "tenant": "061a4443-8d7a-415a-91ad-104e12ff3359"
}
```

# Create a private key

```
ssh-keygen \
    -t rsa \
    -b 4096 \
    -C "azureuser@myserver" \
    -f ~/.ssh/ansible-cluster-key \
    -N mypassphrase
```

# Create the Cluster

## créez un fichier credentials pour Ansible

```
mkdir ~/.azure
vi ~/.azure/credentials
```

Edit the credentials

```
[default]
subscription_id=<your-subscription_id>
client_id=<security-principal-appid>
secret=<security-principal-password>
tenant=<security-principal-tenant>
```

## Edit the YAML

ssh_key value comes from `cat /Users/nicolas/.ssh/ansible-cluster-key.pub`

```
- name: Create Azure Kubernetes Service
  hosts: localhost
  connection: local
  vars:
    resource_group: ansible-rg2
    location: westeurope
    aks_name: ansibleAKSCluster
    username: azureuser
    ssh_key: "..."
    client_id: "0cf730b5-9e49-44a8-b163-2626814d1206"
    client_secret: "1425915e-d539-407d-86e1-c92b68f8724a"
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: "{{ resource_group }}"
      location: "{{ location }}"
  - name: Create a managed Azure Container Services (AKS) cluster
    azure_rm_aks:
      name: "{{ aks_name }}"
      location: "{{ location }}"
      resource_group: "{{ resource_group }}"
      dns_prefix: "{{ aks_name }}"
      linux_profile:
        admin_username: "{{ username }}"
        ssh_key: "{{ ssh_key }}"
      service_principal:
        client_id: "{{ client_id }}"
        client_secret: "{{ client_secret }}"
      agent_pool_profiles:
        - name: default
          count: 2
          vm_size: Standard_D2_v2
      tags:
        Environment: Production`
```


