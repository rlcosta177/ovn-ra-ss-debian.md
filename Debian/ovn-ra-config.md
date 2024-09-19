## OpenVPN Integration | Remote Acces(haven't done yet)

Important:

    In Remote Access VPNs, the clients are aware of the vpn(they initiate the connection to the server themselves) and should be able to see the tunnel when using 'ip a' and 'ifconfig'

Topology:

    Certificate Authority: Lux East(lux-east will operate as the vpn server, but will seperately be the CA as well, so you can think of the CA as being a different machine that will sign the requests of lux-east and lux-west servers)
    VPN Server: Lux East
    Clients: new client for remote access(maybe done in us-east-1? not sure) 

Easy install:
- go to [https://github.com/angristan/openvpn-install]
- `chmod +x openvpn-install.sh`
- `sudo ./openvpn-install.sh`
- follow the prompts
- generate the client .ovpn file and send it to the client
- the vpn should be setup.
- if running on arch, there could be an error however(openssh server fails to start)
  - refer to this link for the fix(antonpetrovmain's comment): [https://github.com/angristan/openvpn-install/issues/1214]

---

Server:
  - the ta.key file should be the same on everyone machine that you install OpenVPN.

remote access:
   - `apt install openvpn -y`
   - `cd /etc/openvpn/`
   - `openvpn --genkey --secret ta.key`
   - `openssl dhparam -out dh2048.pem 2048`
   - `nano server_ra.conf`


 server.conf file example:
 ```bash
 local 192.168.1.2
 port 1194
 proto udp
 dev tun
 ca ca.crt
 cert server.crt
 key server.key
 dh dh.pem
 auth SHA512
 tls-crypt tc.key
 topology subnet
 server 10.8.0.0 255.255.255.0
 push "redirect-gateway def1 bypass-dhcp"
 ifconfig-pool-persist ipp.txt
 push "dhcp-option DNS 1.1.1.1"
 push "dhcp-option DNS 1.0.0.1"
 push "block-outside-dns"
 keepalive 10 120
 user nobody
 group nogroup
 persist-key
 persist-tun
 verb 3
 crl-verify crl.pem
 explicit-exit-notify
 ```

 client.conf file example:

 ```bash
 client
 dev tun
 proto udp
 remote 83.240.144.61 1194
 resolv-retry infinite
 nobind
 persist-key
 persist-tun
 remote-cert-tls server
 auth SHA512
 ignore-unknown-option block-outside-dns
 verb 3

 <ca>
 -----BEGIN CERTIFICATE-----
 -----END CERTIFICATE-----
 </ca>
 <cert>
 -----BEGIN CERTIFICATE-----
 -----END CERTIFICATE-----
 </cert>
 <key>
 -----BEGIN PRIVATE KEY-----
 -----END PRIVATE KEY-----
 </key>
 <tls-crypt>
 -----BEGIN OpenVPN Static key V1-----
 -----END OpenVPN Static key V1-----
 </tls-crypt>
 ```

 directory in which server.conf file is at:
 ```bash
 -rw------- 1 root   root    1204 Sep 19 10:01 ca.crt
 -rw------- 1 root   root    1704 Sep 19 10:01 ca.key
 -rw-r----- 1 root   root     186 Sep 19 10:01 client-common.txt
 -rw------- 1 nobody nogroup  650 Sep 19 10:01 crl.pem
 -rw-r----- 1 root   root     424 Sep 19 10:01 dh.pem
 drwxr-xr-x 5 root   root    4096 Sep 19 10:01 easy-rsa
 -rw------- 1 root   root       0 Sep 19 10:11 ipp.txt
 -rw-r----- 1 root   root     444 Sep 19 10:01 server.conf
 -rw------- 1 root   root    4511 Sep 19 10:01 server.crt
 -rw------- 1 root   root    1704 Sep 19 10:01 server.key
 -rw------- 1 root   root     636 Sep 19 10:01 tc.key
 ```

Ending:
- `systemctl disable openvpn` && `systemctl stop openvpn`
- `systemctl enable --now openvpn@server_ra`
- create a port forwarding rule on your router/firewall for openvpn traffic to flow through
