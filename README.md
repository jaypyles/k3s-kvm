# Creating a k3s cluster using KVM

Install all pre-requisites for kvm to begin:
`sudo apt install kvm`

Create a bridge network on the host:

```bash
# /etc/network/interfaces

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet manual

auto br0
iface br0 inet static
address 192.168.1.111
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.254
dns-nameservers 192.168.1.254 8.8.8.8
bridge_ports eth0
```

And then: `sudo systemctl restart networking`

Create a virtual network using the bridge network: 

```bash
# bridge-network.xml

<network>
  <name>net</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
  <domain name="example"/>
</network>
```

```bash
virsh net-define bridge-network.xml
virsh net-start net
virsh net-autostart net
```

Create two virtual machines using either virt-install or virt-manager. These virtual machines will be connected to the bridge network created above.

The machines will need ip addresses set using netplan if using ubuntu-server.

```bash
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    net:
      addresses:
        - 192.168.1.x/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Add your virtual machines to an `inventory.ini` file:

```bash
[k3s_master]
master ansible_host=192.168.x.x

[k3s_workers]
worker-1 ansible_host=192.168.x.x

[all:vars]
ansible_user=user
ansible_pass=root
```

Create a vars file for the kvm variables:

```bash
cp vars/kvm.template.yml vars/kvm.yml
```

Run the playbook to install k3s on the virtual machines:
`ansible-playbook -i inventory.ini playbooks/create_k3s_cluster.yml`

## Remote Control of k3s cluster

Copy over the k3s.yaml to the host you want to control from:

`scp user@master_node:k3s.yaml ~/.kube/config`

Then, edit the config file to point to the host you want to control:

`vim ~/.kube/config`

And then: `server: https://master_node:6443`

### Set KVM vm's to restart on reboot (optional)

`virsh autostart <vm_name>`