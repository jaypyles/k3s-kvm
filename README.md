# Creating a k3s cluster using KVM

Install all pre-requisites for kvm to begin:
`sudo apt install kvm`

Create a bridge network on the host:

```bash
export BRIDGE="br0"
export MASTER="enp2s0"

sudo ip link add name ${BRIDGE} type bridge
sudo ip link set ${MASTER} master ${BRIDGE}
sudo ip link set ${BRIDGE} up
sudo ip link set ${MASTER} up
```

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

Assign your vms to a static address using netplan:

```bash
# /etc/netplan/01-static-ip.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      addresses:
        - 192.168.x.x/24   # Your static IP and subnet mask
      gateway4: 192.168.x.x  # Your network gateway (adjust if necessary)
      nameservers:
        addresses:
          - 8.8.8.8          # Primary DNS (Google DNS)
          - 8.8.4.4          # Secondary DNS (Google DNS)
```

And then: `sudo netplan apply`

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