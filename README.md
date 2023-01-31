# WireGuardConfig

Spending quite a while researching different guides for setting up your personal VPN on Ubuntu server Using WireGuard, finally decided to create my own ultimate guide, covering all the aspects I faced:

First, installing wireguard on Linux server:

```bash
apt install wireguard
```

Secondly, we need 2 key pairs for client-server communation. Both can be generated on the same server:

```bash
$ wg genkey | tee privatekeysrv | wg pubkey > publickeysrv
$ cat p*

WNnpjGGB4NtGrgR7yoyql/2y/orFvArwYGFYzgD7a2I=
OY/16eY93vtIWkbwrI9dINiET5OthmZqdxvv0lOmwVw=

$ wg genkey | tee privatekeycl | wg pubkey > publickeycl
$ cat p*

eEL5XZQuyj9ffsF53+ia/ApKvQ1VM5+0Hzlu42iFymo=
zurCAgbi+mOq/JNIBN8FwQedF53V15Ahp7erhgdZWxM=
```
Then creating config files on server and client. 
The structure of the configs:

Server:

```bash
sudo nano /etc/wireguard/wg0.conf

[Interface]
#Server's private key: privatekeysrv
PrivateKey = WNnpjGGB4NtGrgR7yoyql/2y/orFvArwYGFYzgD7a2I=
#VPN network - should be different from the current netw
Address = 192.168.20.1/24
#Forwarding rule for traffic from wg0 to eth0 and back
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
#connection port
ListenPort = 51820

[Peer]
#Public key of Client: publickeycl
PublicKey = DHDsdaE9Vs1HJyLJ5/jsdZdFzg+aq6GL7+bMkKsu4DA=
#IP address of the connecting client
AllowedIPs = 192.158.20.2/32
```

Client (save as config and import to any client, e.g. Wireguard for Mac):

```bash
[Interface]
# Client's Private key: privatekeycl
PrivateKey = eEL5XZQuyj9ffsF53+ia/ApKvQ1VM5+0Hzlu42iFymo=
#Address of the client in VPN network
Address = 192.168.20.2/24
#DNS server
DNS = 1.1.1.1, 8.8.8.8

[Peer]
#Servers public key: publickeysrv
PublicKey = OY/16eY93vtIWkbwrI9dINiET5OthmZqdxvv0lOmwVw=
#will forward all traffic over the WireGuard VPN connection. 
#If you want to only use WireGuard for specific destinations, set IP address ranges in the list separated by a comma.
AllowedIPs = 0.0.0.0/0
#Public IP of the server + port
Endpoint = 165.232.82.165:51820
```

Once server conf is set up, start wg:

`sudo wg-quick up wg0`

or as a service:

`sudo systemctl start wg-quick@wg0`

To enable WireGuard to start automatically at system boot, also enable the systemd service.

`systemctl enable wg-quick@wg0`

Depending on the VPS it might be needed to set up IPv4 forwarding:


``` bash
$ sudo nano /etc/sysctl.conf

#add the following line 

net.ipv4.ip_forward=1

$ sudo sysctl -p
```

Once the client is connected to the server, wg output supposes to have handshake details:
``` bash 
wg
interface: wg0
  public key: ***
  private key: (hidden)
  listening port: 51820

peer: ***
  endpoint: Client's public IP
  allowed ips: 192.168.20.2/32
  latest handshake: 33 seconds ago
  transfer: 9.24 MiB received, 9.42 MiB sent
  ```
