# 5 Dakikada site-to-site VPN (Wireguard)
## Server
### DEB
```bash
sudo apt update 
sudo apt install wireguard -y 
```
### RPM
```bash
sudo yum install epel-release elrepo-release 
sudo yum install kmod-wireguard wireguard-tools
```
## Keys
```bash
wg genkey | tee privatekey | wg pubkey > publickey
```
## Config
```bash
sudo vim /etc/wireguard/wg0.conf
```
```
[Interface]  
Address = 10.0.0.1/32  
ListenPort = 51820  
PrivateKey = <YOUR-SERVER-PRIVATE-KEY-STRING>
  
[Peer]  
PublicKey = <YOUR-CLIENT-PUBLIC-KEY-STRING>
AllowedIPs = 10.0.0.0/24
```
### ipv4.ip_forward
```bash
sudo vim /etc/sysctl.conf
```
```
net.ipv4.ip_forward=1
```
```
sudo sysctl -p
```
## Start Server
```bash
wg-quick up /etc/wireguard/wg0.conf
```
## Client
### DEB
```bash
sudo apt update 
sudo apt install wireguard -y 
```
### RPM
```bash
sudo yum install epel-release elrepo-release 
sudo yum install kmod-wireguard wireguard-tools
```
## Keys
```bash
wg genkey | tee privatekey | wg pubkey > publickey
```
## Config
```
sudo vim /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 10.0.0.2/32
ListenPort = 51820
PrivateKey = <YOUR-CLIENT-PRIVATE-KEY>

[Peer]
PublicKey = <YOUR-SERVER-PUBLIC-KEY>
Endpoint = <SERVER-PUBLIC-IP-ADDRESS>:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```
## Start Client
```
wg-quick up /etc/wireguard/wg0.conf
```
