# [Advanced] Setting up bridge interface between two VM, Part 01

Created by: Raihan Islam
Created time: March 18, 2024 1:17 PM
Tags: Engineering, Networking

<aside>
💡 This documentation will guide you through setting up a bridge interface between two virtual machines (VMs). The process is divided into three main parts: Host setup, VM launch and tap device configuration, and VM interaction setup.

</aside>

## **Part 1: Host Setup**

> In this part, we prepare the host machine for hosting the bridge interface and facilitating communication between the VMs and the external network. We create a bridge interface, assign an IP address to it, bring it up, and configure firewall rules to enable outbound traffic from the VMs.
> 

1. **Creating a Bridge Interface:**

```bash
sudo ip link add name br0 type bridge
```

1. **Assign IP Address:**

```bash
sudo ip addr add 192.168.1.7/24 dev br0
```

1. **Bring Up the Interface:**

```bash
sudo ip link set dev br0 up
```

1. **Configure Firewall Rules:**

```bash
sudo iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE
```

- This command sets up a NAT rule that masquerades outgoing packets from the specified outgoing interface, allowing the VMs connected to the bridge interface to communicate with external networks by translating their internal IP addresses to the external IP address of the host machine.

![bridge-interface.png](%5BAdvanced%5D%20Setting%20up%20bridge%20interface%20between%20two%20b46cb38ea2bf4215b912e695e314c72a/bridge-interface.png)

## **Part 2: VM Launch and Tap Device Configuration**

> This part focuses on configuring the virtual machines and establishing connectivity with the bridge interface. We ensure the absence of conflicting tap devices, create a new tap device if necessary, bring it up, and then attach it to the bridge interface to enable communication with the VMs.
> 

1. **Check and Create Tap Device:**

```bash
sudo ip link del "$TAP_DEV" 2> /dev/null || true
sudo ip tuntap add dev "$TAP_DEV" mode tap
```

1. **Bring Up Tap Interface:**

```bash
sudo ip link set dev "$TAP_DEV" up
```

1. **Attach Tap Device to Bridge Interface:**

```bash
sudo ip link set dev $TAP_DEV master $BRIDGE
```

- Before launching the VM, ensure that there is no existing tap device with the desired name. If there is, delete and recreate it. Then run the `VM Launch Script` . For now we will skip it.

![tap-config.png](%5BAdvanced%5D%20Setting%20up%20bridge%20interface%20between%20two%20b46cb38ea2bf4215b912e695e314c72a/tap-config.png)

## **Part 3: VM Interaction Setup**

> The final part involves configuring the network interfaces within the VMs for communication with the bridge interface. We assign an IP address to the VM's interface (`eth0`), bring it up, and add routing rules if necessary. Finally, we verify the connectivity between the VM and the bridge interface to ensure successful communication.
> 

1. **Assign IP Address to VM Interface:**

```bash
sudo ip addr add 192.168.1.169/24 dev eth0
```

1. **Bring Up VM Interface:**

```bash
sudo ip link set dev eth0 up
```

1. **Add Routing Rules:**

```bash
sudo ip r add 192.168.1.1 via 192.168.1.7 dev eth0
sudo ip r add default via 192.168.1.7 dev eth0
```

![vm-01.png](%5BAdvanced%5D%20Setting%20up%20bridge%20interface%20between%20two%20b46cb38ea2bf4215b912e695e314c72a/vm-01.png)

1. **Verification:**

```bash
ping 192.168.1.7
```

> By following these steps, we can configure another VM with a different tap device to connect to the same bridge interface, allowing multiple VMs to communicate with each other and the external network through the host machine.
> 

![vm-02.png](%5BAdvanced%5D%20Setting%20up%20bridge%20interface%20between%20two%20b46cb38ea2bf4215b912e695e314c72a/vm-02.png)