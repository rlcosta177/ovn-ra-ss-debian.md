# Routing with ubuntu + aws

Topology:

    us-east-1 -> [1 server(lux-server-east), 1 client(lux-cli-east)]  | 172.31.96.0/20   | lux-srv is the site-to-site server(local)
    us-west-2 -> [1 server(lux-server-west), 1 client(lux-cli-west)]  | 172.31.112.0/20  | lux-srv is the site-to-site client(remote)


---
EAST SERVER NICs: 
    
    172.31.0.100, 172.31.96.100(all /20)

EAST DMZ/CLIENT NIC:
    
    172.31.96.101

---

WEST SERVER NICs:

    172.31.0.100, 172.31.112.100(all /20)

WEST LUX-CLIENT NIC:
    
    172.31.112.101

WEST WIN-CLIENT NIC:
    
    172.31.112.102

---

## Setting up Routing

---

1) Enable routing on AWS(on a single instance):
    - in AWS -> click the server -> actions -> networking -> change source/destination check -> check the box 'stop'


2) in the server(enabling forwarding so that the server can act as a router):
    - `sudo apt update && sudo apt upgrade -y`
    - `sudo apt install netfilter-persistent iptables-persistent -y`
    - `nano /etc/sysctl.conf` -> uncomment: net.ipv4.ip_forward=1
    - `iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE` (routes any incoming traffic(from clients) to the internet, through if ens5)
    - `netfilter-persistent save`
    - `sysctl -p <- applies the changes` 
    - `systemctl restart iptables`

    <details>
      <summary>reference for prerouting, input, forward, output, postrouting</summary>
        
        https://pastebin.com/SxhJmhrm
        https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture
    </details>


    <details>
      <summary>reference for the NAT and FILTER tables(jdaniel email)</summary>
        
        (substituir os tracos('-'), esses nao funcionam)
        practical examples: https://pastebin.com/7D90FwF5
        specific examples: https://pastebin.com/dLYVkAaS
      </details>

3) in the server(add a dns forwarder to 1.1.1.1 to redirect unknown requests the local dns doesn't know to that IP):
    - `sudo apt install bind9 bind9-utils bind9-dnsutils bind9-doc`
    - `sudo nano /etc/bind/named.conf.options`  (ref: https://pastebin.com/W4ibnbVW)
    - `sudo systemctl restart bind9`

4) in the clients(DNS & ROUTES):
    - `sudo nano /etc/netplan/50-cloud-init.yaml`  (ref: https://pastebin.com/wh3PKFrV) | (full ref: https://pastebin.com/uxBEM3mg) |  'nameservers' tem que estar na mesma linha que dhcp4-overrides
    - `sudo netplan try dry` (try config without making changes)
    - `netplan try`
    - `netplan apply`
    - `sudo systemctl restart systemd-networkd`

5) enable rdp on win-inside:
    - Para poder dar rdp da maquina fisica para o win-inside, tenho que dar rdp a partir do lux-inside(172.31.112.101) para o win-inside(172.31.112.102) e alterar o ip, mask, gateway e dns no win-inside

7) in the server(create the nat policies):
    - only do this if you want to do port forwarding(redirect requests to another ip based on the port)
    - https://pastebin.com/MWLpsXu8

8)  if connectivity to the internet isn't working on the client(amazon linux only i think):
       - use `route -n` to see where the traffic goes through
       - if the gateway is not the one we want, use this command:
            - `sudo route del default gw 172.31.112.1`(wrong gateway)
            - `sudo route add default gw 172.31.112.100`(gateway we want)
       - https://gist.github.com/jdmedeiros/0b6208d6e0a7cf35d31f5749be47d8a2 <- the 80-ec2.network file is the same as netplan, if the other settings didn't work, its because they are ignored and only 80-ec2.network will make changes to the routing options of the client

9) IMPORTANT FILES & COMANDS:
    - `tcpdump -i interface [src/dst host/port ip/port]` | very useful to troubleshoot connectivity issues
    - in the client, linux amazon  /etc/sysconfig/network-scripts/ (ifcfg-eth0 && route-eth0) | outdated i believe, changes here won't take effect depending on the version
    - in the client, ubuntu /etc/netplan/50-cloud-init.yaml
    - in the server, ubuntu&linux amazon /etc/sysctl.conf (uncomment 'net.ipv4.ip_forward=1')
    - in the server, run `sysctl -p` after changing /etc/sysctl.conf
    - in the server `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE` <- necessary if u want a client attached to an interface of the server to access the internet
    - `netfilter-persistent save` OR reload <- after doing any iptables change(save if in the cli | reload if in the rules.v4 file)
    - `route -n` OR `ip route` to see the routing table, useful if having problems accessing the internet with the client
