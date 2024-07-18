# Network Configuration Management on Amazon Linux 2023

This guide details how to override the default gateway assigned by DHCP.

## Opening and Editing Network Configuration

1. **Open the Network Configuration File**
   Use `nano` (or any text editor of your choice) to open the network configuration file:
   ```bash
   sudo nano /usr/lib/systemd/network/80-ec2.network
   ```
   to add the **Gateway=172.31.96.100** line.


2. **View the Network Configuration File Content**
   After editing, you can view the contents to confirm the changes:
   ```bash
   cat /usr/lib/systemd/network/80-ec2.network
   ```
   
## Network Configuration File Content

Below is an example of the content inside the `80-ec2.network` file:
```
[Match]
Driver=ena ixgbevf vif

[Link]
MTUBytes=9001

[Network]
DHCP=yes
IPv6DuplicateAddressDetection=0
LLMNR=no
DNSDefaultRoute=yes
Gateway=172.31.96.100

[DHCPv4]
UseHostname=no
UseDNS=yes
UseNTP=yes
UseDomains=yes

[DHCPv6]
UseHostname=no
UseDNS=yes
UseNTP=yes
WithoutRA=solicit
```

## Viewing Current Route Table

3. **Check the Current Route Table**
   Before restarting the network service, check the current route table:
   ```bash
   route -n
   ```
   Expected Output:
   ```
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   0.0.0.0         172.31.96.1     0.0.0.0         UG    512    0        0 enX0
   172.31.0.2      172.31.96.1     255.255.255.255 UGH   512    0        0 enX0
   172.31.96.0     0.0.0.0         255.255.240.0   U     512    0        0 enX0
   172.31.96.1     0.0.0.0         255.255.255.255 UH    512    0        0 enX0
   ```

## Restarting Network Service

4. **Restart the Network Service**
   Apply any changes made by restarting the `systemd-networkd` service:
   ```bash
   sudo systemctl restart systemd-networkd
   ```

## Verifying Changes

5. **Check the Updated Route Table**
   After restarting, verify that the changes are effective:
   ```bash
   route -n
   ```
   Updated Output:
   ```
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   0.0.0.0         172.31.96.100   0.0.0.0         UG    0      0        0 enX0
   0.0.0.0         172.31.96.1     0.0.0.0         UG    512    0        0 enX0
   172.31.0.2      172.31.96.1     255.255.255.255 UGH   512    0        0 enX0
   172.31.96.0     0.0.0.0         255.255.240.0   U     512    0        0 enX0
   172.31.96.1     0.0.0.0         255.255.255.255 UH    512    0        0 enX0
   ```

## Testing Connectivity

6. **Ping Test**
   Finally, perform a ping test to confirm network connectivity:
   ```bash
   ping -c3 www.google.com
   ```
   Output:
   ```
   PING www.google.com (172.253.62.99) 56(84) bytes of data.
   64 bytes from bc-in-f99.1e100.net (172.253.62.99): icmp_seq=1 ttl=104 time=2.75 ms
   64 bytes from bc-in-f99.1e100.net (172.253.62.99): icmp_seq=2 ttl=104 time=2.30 ms
   64 bytes from bc-in-f99.1e100.net (172.253.62.99): icmp_seq=3 ttl=104 time=2.48 ms

   --- www.google.com ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2003ms
   rtt min/avg/max/mdev = 2.298/2.506/2.747/0.184 ms
   ```
