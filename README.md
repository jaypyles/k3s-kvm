# Creating a k3s cluster using KVM

Install all pre-requisites for kvm to begin:
`sudo apt install kvm`

Assign your vms to a static address using netplan:

```bash
# /etc/network/interfaces

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet manual

auto brx
iface brx inet static
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

Run the playbook to install k3s on the virtual machines:
`ansible-playbook -i inventory.ini playbooks/create_k3s_cluster.yml`