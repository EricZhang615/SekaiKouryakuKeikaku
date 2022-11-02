# Deploy Multinode OpenStack VIM via Kolla Ansible

## Environment

* 3 physic servers with 2 nic each
* Ubuntu18.04 LTS
* Openstack Victoria

## Preparation

After installed Ubuntu, change the source of apt for better connectivity:
```(bash)
sudo sed -i "s@http://.*archive.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
sudo sed -i "s@http://.*security.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list

sudo apt update

sudo apt install net-tools
```

### Virtual environment

```
sudo apt install python3-venv
python3 -m venv /path/to/venv
source /path/to/venv/bin/activate
```

### Docker

[Official Guide](https://docs.docker.com/engine/install/ubuntu/)

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
    ```
    sudo apt-get update
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```

2. Add Docker’s official GPG key:
    ```
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

3. Use the following command to set up the repository:
    ```
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

4. Update the apt package index:
    ```
    sudo apt-get update
    ```
    >Receiving a GPG error when running apt-get update?
    >```
    >sudo chmod a+r /etc/apt/keyrings/docker.gpg
    >sudo apt-get update
    >```

5. Install Docker Engine, containerd, and Docker Compose.
    ```
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```

6. (Optional) Verify that the Docker Engine installation is successful by running the hello-world image:
    ```
    sudo docker run hello-world
    ```

7. Install docker package for python
    ```
    (venv) $ pip install docker
    ```

## Prepare Kolla-ansible

[Official Quick Start](https://docs.openstack.org/kolla-ansible/victoria/user/quickstart.html)

[OpenStack VIM Installation Guide](https://docs.openstack.org/tacker/victoria/install/openstack_vim_installation.html)

### Install Kolla-ansible

1. Install Ansible. **As Victoria**, Kolla Ansible requires at least Ansible 2.9 and supports up to 2.9.
    ```
    pip install -U pip
    pip install 'ansible<2.10'
    ```

2. Install kolla-ansible and its dependencies using pip.
    ```
    pip install git+https://opendev.org/openstack/kolla-ansible@stable/victoria
    ```

3. Create the `/etc/kolla` directory.
    ```
    sudo mkdir -p /etc/kolla
    sudo chown $USER:$USER /etc/kolla
    ```

4. Copy `globals.yml` and `passwords.yml` to `/etc/kolla` directory.
    ```
    cp -r /path/to/venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
    ```

5. Copy `all-in-one` and `multinode` inventory files to the current directory. In this guide, we will use multinode for deployment but `all-in-one` file is also needed when pulling images during the setup of local docker registry.
    ```
    cd ~/
    cp /path/to/venv/share/kolla-ansible/ansible/inventory/* .
    ```

6. Configure Ansible
    >For best results, Ansible configuration should be tuned for your environment. For example, add the following options to the Ansible configuration file `/etc/ansible/ansible.cfg`:
    >```
    >[defaults]
    >host_key_checking=False
    >pipelining=True
    >forks=100
    >```

## Prepare local docker registry

It is better to set up a local docker registry in multinode deployment, so that other nodes could pull images from local net rather than Internet. 

### Configure `globals.yml`
For VIM Installation, an example is given below:
```(YAML)
---
kolla_install_type: "source"
kolla_internal_vip_address: "<an unused IP as float IP>"
docker_registry: "10.1.0.6:4000"
docker_namespace: "lokolla"
api_interface: "eth0"
tunnel_interface: "eth1"
neutron_external_interface: "eth2"
enable_glance: "yes"
enable_haproxy: "yes"
enable_keystone: "yes"
enable_mariadb: "yes"
enable_memcached: "yes"
enable_neutron: "yes"
enable_nova: "yes"
enable_rabbitmq: "yes"
enable_aodh: "yes"
enable_ceilometer: "yes"
enable_gnocchi: "yes"
enable_heat: "yes"
enable_horizon: "yes"
enable_neutron_sfc: "yes"
```

