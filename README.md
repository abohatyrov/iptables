# Redirect traffic from port 81 in one subnetwork to port 80 in another subnetwork (Using Centos/RHEL 6)
## Introduction
This guide will show you how to redirect traffic from port 81 in one subnetwork to port 80 in another subnetwork using NAT prerouting and postrouting on Centos/RHEL 6.

## Prerequisites 
- Two virtual machines with CentOS/RHEL 6
- Installed `iptables`, `curl` and `httpd` packages

## A little bit of theory
Port forwarding allows you to forward incoming packets to another destination, such as another port or IP address. This can be useful for redirecting traffic to a specific server or device on your network.

To set up port forwarding, you need to configure your firewall to allow incoming connections on the port that you want to forward, and then create a rule to tell the firewall where to forward the traffic.

__Here is a simplified explanation of how port forwarding works:__

1. A client sends a packet to port 81 on server A.
2. Server A's firewall intercepts the packet and checks to see if there is a port forwarding rule for port 81.
3. If there is a port forwarding rule for port 81, the firewall modifies the packet's destination port to port 80 and forwards the packet to server B.
4. Server B receives the packet and processes it as if it had been sent directly to port 80.
5. Server B sends a response packet back to the client.
6. The firewall on server A intercepts the response packet and modifies the packet's source port to port 81 before forwarding the packet to the client.
7. The client receives the response packet and thinks that it came from server A on port 81.

To set up port forwarding, you need to configure your firewall and create a rule to tell it where to forward the traffic.

### Remark about iptables
In CentOS/RHEL 7 iptables has been replaced with firewalld. To use iptables instead of firewalld on CentOS/RHEL 7, you need to disable firewalld and enable iptables instead. In this exmple I am using CentOS 6, so I do not need to do that.

## Preparations
In this step we will prepare the system for NAT. You can use provided commands on your CentOS 6 or RHEL 6 machines. This step must be done on both machines.

### IP Forwarding
```bash
[abohat@vm ~]$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```
If the output is equal to 0, this means that IP forwarding is disabled. To enable IP forwarding use provided command:

```bash
[abohat@vm ~]$ sudo echo "net.ipv4.ip_forward = 1"|sudo tee /etc/sysctl.d/99-ipforward.conf
net.ipv4.ip_forward = 1
```

And to activate the changes:
```bash
[abohat@vm ~]$ sudo sysctl -p /etc/sysctl.d/99-ipforward.conf
net.ipv4.ip_forward = 1
```

### Iptables
Next you need to check that ip tables is running on your system. Iptables is running as a kernel module, so it can't be seen as one of the normal processes. You need to use next command:
```bash
[abohat@vm ~]$ lsmod|grep iptable
iptable_nat            12928  0
nf_nat                 18242  1 iptable_nat
nf_conntrack_ipv4      14078  3 nf_nat,iptable_nat
nf_conntrack           52720  3 nf_conntrack_ipv4,nf_nat,iptable_nat
iptable_filter         12536  0
ip_tables              22042  2 iptable_filter,iptable_nat
x_tables               19118  3 ip_tables,iptable_filter,iptable_nat
```

If there is no output, it means that iptables is disabled.
To start iptables use provided command:
```bash
[abohat@vm ~]$ sudo systemctl start iptables
```





