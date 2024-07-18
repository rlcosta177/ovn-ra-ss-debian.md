## OpenVPN Integration | Remote Acces(haven't done yet)

Important:

    In Remote Access VPNs, the clients are aware of the vpn(they initiate the connection to the server themselves) and should be able to see the tunnel when using 'ip a' and 'ifconfig'

Topology:

    Certificate Authority: Lux East(lux-east will operate as the vpn server, but will seperately be the CA as well, so you can think of the CA as being a different machine that will sign the requests of lux-east and lux-west servers)
    VPN Server: Lux East
    Clients: new client for remote access(maybe done in us-east-1? not sure) 

References:

    USE THIS: https://github.com/JoseCarvalho1026/VPN/blob/main/Enta/server_ra.md
    client&server.conf templates: https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files

Test this(from: jaime-10-reis):

Server:
  - the ta.key file should be the same on everyone machine that you install OpenVPN.

remote access:
   - `apt install openvpn -y`
   - `cd /etc/openvpn/`
   - `openvpn --genkey --secret ta.key`
   - `openssl dhparam -out dh2048.pem 2048`
   - `nano server_ra.conf`

example:
```bash
port 1194
proto udp
dev tun
ca ca.crt
cert vpn.inova.pt.crt
key vpn.inova.pt.key
dh dh2048.pem
server 10.10.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt
push "route 10.0.100.0 255.255.255.0"
keepalive 10 120
tls-auth ta.key 0
cipher AES-256-CBC
persist-key
persist-tun
verb 3
explicit-exit-notify 1
```

➜ `systemctl disable openvpn` && `systemctl stop openvpn`
➜ `systemctl enable --now openvpn@server_ra`

