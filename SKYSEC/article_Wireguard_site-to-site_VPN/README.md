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
```bash
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
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

```
wg set wg0 peer <client_pubkey> allowed-ips 10.0.0.x/32
```

```
wg-quick save wg0
```

# Peer Scripti
Yeni kullanıcı eklemek için otomasyon scripti

```bash
#!/bin/bash


if [ -z "$1" ]; then
  echo "Kullanıcı adını yaz"
  exit 1
fi
if [ -z "$2" ]; then
  echo "IP adresini yaz"
  exit 1
fi

# Sample Public Key
server_public_key='server public key '

# Wireguard Server IP or Domain
server_ip_or_domain='server ip or domain'

# Interface name is the same as the name of your Wireguard configuration file
interface_name="skysecVpn"

root=/var/wireguard/keys/$1
ip=$2
$subnet="192.168.231.0/24"

mkdir -p $root
wg genkey | tee $root/privatekey | wg pubkey > $root/publickey
echo "[Interface]" | tee -a $root/$interface_name.conf
echo "Address = $ip/32" | tee -a $root/$interface_name.conf
echo "ListenPort = 51820" | tee -a $root/$interface_name.conf
echo "PrivateKey = $(cat $root/privatekey)" | tee -a $root/$interface_name.conf

echo "[Peer]" | tee -a $root/$interface_name.conf
echo "PublicKey = $server_public_key" | tee -a $root/$interface_name.conf
echo "Endpoint = $server_ip_or_domain:51820" | tee -a $root/$interface_name.conf
echo "AllowedIPs = $subnet" | tee -a $root/$interface_name.conf
echo "PersistentKeepalive = 25" | tee -a $root/$interface_name.conf


wg set $interface_name peer $(cat $root/publickey) allowed-ips $ip/32 persistent-keepalive 25
```

```bash
bash script.py client-name 10.0.0.x
```
