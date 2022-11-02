# Deploy Multinode OpenStack VIM via Kolla Ansible

## Environment

* 3 physic servers with 2 nic each
* Ubuntu18.04 LTS
* Openstack Victoria
* All of the commands are executed under root permission

## Preparation

After installed Ubuntu, change the source of apt for better connectivity:
```bash
sudo sed -i "s@http://.*archive.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
sudo sed -i "s@http://.*security.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list

sudo apt update

sudo apt install net-tools
```

### Virtual environment

```bash
sudo apt install python3-venv
python3 -m venv /path/to/venv
source /path/to/venv/bin/activate
```

### Docker

[Official Guide](https://docs.docker.com/engine/install/ubuntu/)

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
    ```bash
    sudo apt-get update
    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```

2. Add Docker’s official GPG key:
    ```bash
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

3. Use the following command to set up the repository:
    ```bash
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

4. Update the apt package index:
    ```bash
    sudo apt-get update
    ```
    >Receiving a GPG error when running apt-get update?
    >```bash
    >sudo chmod a+r /etc/apt/keyrings/docker.gpg
    >sudo apt-get update
    >```

5. Install Docker Engine, containerd, and Docker Compose.
    ```bash
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```

6. (Optional) Verify that the Docker Engine installation is successful by running the hello-world image:
    ```bash
    sudo docker run hello-world
    ```

7. Install docker package for python
    ```bash
    (venv) $ pip install docker
    ```

## Prepare Kolla-ansible

[Official Quick Start](https://docs.openstack.org/kolla-ansible/victoria/user/quickstart.html)

[OpenStack VIM Installation Guide](https://docs.openstack.org/tacker/victoria/install/openstack_vim_installation.html)

### Install Kolla-ansible

1. Install Ansible. **As Victoria**, Kolla Ansible requires at least Ansible 2.9 and supports up to 2.9.
    ```bash
    pip install -U pip
    pip install 'ansible<2.10'
    ```

2. Install kolla-ansible and its dependencies using pip.
    ```bash
    pip install git+https://opendev.org/openstack/kolla-ansible@stable/victoria
    ```

3. Create the `/etc/kolla` directory.
    ```bash
    sudo mkdir -p /etc/kolla
    sudo chown $USER:$USER /etc/kolla
    ```

4. Copy `globals.yml` and `passwords.yml` to `/etc/kolla` directory.
    ```bash
    cp -r /path/to/venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
    ```

5. Copy `all-in-one` and `multinode` inventory files to the current directory. In this guide, we will use multinode for deployment but `all-in-one` file is also needed when pulling images during the setup of local docker registry.
    ```bash
    cd ~/
    cp /path/to/venv/share/kolla-ansible/ansible/inventory/* .
    ```

6. Configure Ansible
    >For best results, Ansible configuration should be tuned for your environment. For example, add the following options to the Ansible configuration file `/etc/ansible/ansible.cfg`:
    >```YAML
    >[defaults]
    >host_key_checking=False
    >pipelining=True
    >forks=100
    >```

## Prepare local docker registry

It is better to set up a local docker registry in multinode deployment, so that other nodes could pull images from local net rather than Internet. 

### Configure `globals.yml`

>**Note:** The following sample is **not** the final settings. 

```YAML
---
kolla_install_type: "source"
kolla_internal_vip_address: "<an unused IP as float IP>"
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

### Pull related images

```bash
(venv) $ kolla-ansible -i all-in-one pull
```

>**Note:** this command will using inventory file `all-in-one` to pull the images on current host, rather than all of the node servers.

### Launch local Docker Registry

```bash
docker run -d \
    --name registry \
    --restart=always \
    -p 4000:5000 \
    -v registry:/var/lib/registry \
    registry:2
```
To avoid port conflict with Keystone service which also using port 5000, we use port 4000 for local image registry. 

After that, you can check the local registry is available or not:
```bash
curl -X GET http://<your-local-ip-addr>:4000/v2/_catalog | python -m json.tool
```
return:
```json
{
    "repositories": []
}
```

If you received an error about HTTPS like:
```
Error:server gave HTTP response to HTTPS client
```
you need to add this address to insecure registries:

* edit `/etc/docker/daemon.json`; if file not exists, creat it;
* add key-values pair like this:
    ```json
    {
        "insecure-registries":[
            "<your-local-ip-addr>:4000"
        ]
    }
    ```
* run
    ```bash
    systemctl daemon-reload
    service docker restart
    ```
* repeat above steps on **each** node

Use this shell script to upload images to local registry:
```bash
#!/usr/bin/env bash

registry_ip='<your-local-ip-addr>'
registry_port='4000'
openstack_version_tag='victoria'

images=$(docker images | grep -v registry | grep -v REPOSITORY | grep -v $registry_ip | awk '{print $1}')

for image in $images;
do
    docker image tag $image:$openstack_version_tag $registry_ip:$registry_port/$image:$openstack_version_tag
done

images=$(docker images | grep $registry_ip | awk '{print $1}')

for image in $images;
do
    docker push $image:$openstack_version_tag
done
```

Now use curl to check the registry again, you may get the return like this:
```json
{
    "repositories": [
        "kolla/ubuntu-source-aodh-api",
        "kolla/ubuntu-source-aodh-evaluator",
        "kolla/ubuntu-source-aodh-listener",
        "kolla/ubuntu-source-aodh-notifier",
        "kolla/ubuntu-source-ceilometer-central",
        "kolla/ubuntu-source-ceilometer-compute",
        "kolla/ubuntu-source-ceilometer-notification",
        "kolla/ubuntu-source-chrony",
        "kolla/ubuntu-source-cron",
        "kolla/ubuntu-source-fluentd",
        "kolla/ubuntu-source-glance-api",
        "kolla/ubuntu-source-gnocchi-api",
        "kolla/ubuntu-source-gnocchi-metricd",
        "kolla/ubuntu-source-haproxy",
        "kolla/ubuntu-source-heat-api",
        "kolla/ubuntu-source-heat-api-cfn",
        "kolla/ubuntu-source-heat-engine",
        "kolla/ubuntu-source-horizon",
        "kolla/ubuntu-source-keepalived",
        "kolla/ubuntu-source-keystone",
        "kolla/ubuntu-source-keystone-fernet",
        "kolla/ubuntu-source-keystone-ssh",
        "kolla/ubuntu-source-kolla-toolbox",
        "kolla/ubuntu-source-mariadb-clustercheck",
        "kolla/ubuntu-source-mariadb-server",
        "kolla/ubuntu-source-memcached",
        "kolla/ubuntu-source-neutron-dhcp-agent",
        "kolla/ubuntu-source-neutron-l3-agent",
        "kolla/ubuntu-source-neutron-metadata-agent",
        "kolla/ubuntu-source-neutron-openvswitch-agent",
        "kolla/ubuntu-source-neutron-server",
        "kolla/ubuntu-source-nova-api",
        "kolla/ubuntu-source-nova-compute",
        "kolla/ubuntu-source-nova-conductor",
        "kolla/ubuntu-source-nova-libvirt",
        "kolla/ubuntu-source-nova-novncproxy",
        "kolla/ubuntu-source-nova-scheduler",
        "kolla/ubuntu-source-nova-ssh",
        "kolla/ubuntu-source-openvswitch-db-server",
        "kolla/ubuntu-source-openvswitch-vswitchd",
        "kolla/ubuntu-source-placement-api",
        "kolla/ubuntu-source-rabbitmq"
    ]
}
```

## Install OpenStack

### Configure `globals.yml` again

The following values should be set properly:
```YAML
ansible_python_interpreter: "/path/to/venv/bin/python"

kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_tag_suffix: ""
kolla_internal_vip_address: "<an unused IP as float IP>"

docker_registry: "<your-local-ip-addr>:4000"

network_interface: "<nic1>"
neutron_external_interface: "<nic2>"

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
>**Note:** if using virtual environment, `ansible_python_interpreter` has to be set correctly. 

### Generate system passwords

```bash
(venv) $ sudo kolla-genpwd
```

### Configure inventory file

You can check the detailed format in Official Guide.
```ini
[control]
<nodes-hostname> ansible_user=username ansible_password=password

[network:children]
control

[compute:children]
control

[monitoring:children]
control
...
```

Check whether the configuration of inventory is correct or not, run:
```bash
(venv) $ ansible -i multinode all -m ping
```
>**Note:** You may get an error related to sshpass, just install it.

### Eventually, Deploy

Pull all of the images to each node:
```bash
(venv) $ kolla-ansible -i multinode pull
```

To install OpenStack system:
```bash
(venv) $ kolla-ansible -i multinode deploy
```

>**Note:** Execute the command with `-vvv` argument to active verbose option is recommended for debugging. 

If you are following official documents, there are another steps before deployment:
```bash
kolla-ansible -i multinode bootstrap-servers
kolla-ansible -i multinode prechecks
```
These steps will help you to do the above configurations, but I always receive the apt mudule not exist problem via virtual environment. Whats more, these steps may break your docker registry GPG key settings. Therefore, you can just skip these steps and deploy it directly, as long as you can make sure the basic environment is set up properly. 

If the deploy tasks are finished with out fails, run:
```bash
(venv) $ kolla-ansible post-deploy
```
With this command, the admin-openrc.sh will be generated at /etc/kolla/admin-openrc.sh.