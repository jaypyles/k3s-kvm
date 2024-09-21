# Creating a k3s cluster using KVM

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

Assign your vm to a static address using netplan:

```bash
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