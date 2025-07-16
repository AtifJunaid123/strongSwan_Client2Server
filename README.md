
# Site-to-Site IPsec VPN with StrongSwan

## Network Topology

```
Client1 (192.168.10.2)
     |
     | LAN
     |
Strongswan1
  ens192: 192.168.10.1
  ens224: 10.0.0.1  <----- TUNNEL ----->  
                                        |
                                    Strongswan2
                                    ens192: 10.0.0.2
                                    ens224: 192.168.20.1
                                        |
                                        | LAN
                                        |
                                    Client2 (192.168.20.2)
```
## Install Strongswan
```bash
sudo apt update
sudo apt install strongswan strongswan-pki libcharon-extra-plugins libcharon-extauth-plugins libstrongswan-extra-plugins
```

## IP Configuration

### Strongswan1
```bash
ip addr add 10.0.0.1/24 dev ens224
ip addr add 192.168.10.1/24 dev ens192
ip link set ens224 up
ip link set ens192 up
```

### Strongswan2
```bash
ip addr add 10.0.0.2/24 dev ens192
ip addr add 192.168.20.1/24 dev ens224
ip link set ens192 up
ip link set ens224 up
```
## /etc/ipsec.conf Configurations

### On Strongswan1:
```ini
config setup
    charondebug="ike 1, knl 1, cfg 0"
    uniqueids=no

conn site-to-site
    auto=start
    type=tunnel
    keyexchange=ikev2

    left=10.0.0.1
    leftid=@strongswan1
    leftsubnet=192.168.10.0/24
    leftauth=psk

    right=10.0.0.2
    rightid=@strongswan2
    rightsubnet=192.168.20.0/24
    rightauth=psk

    ike=aes256-sha256-modp2048
    esp=aes256-sha256
```

### On Strongswan2:
```ini
config setup
    charondebug="ike 1, knl 1, cfg 0"
    uniqueids=no

conn site-to-site
    auto=start
    type=tunnel
    keyexchange=ikev2

    left=10.0.0.2
    leftid=@strongswan2
    leftsubnet=192.168.20.0/24
    leftauth=psk

    right=10.0.0.1
    rightid=@strongswan1
    rightsubnet=192.168.10.0/24
    rightauth=psk

    ike=aes256-sha256-modp2048
    esp=aes256-sha256
```


## /etc/ipsec.secrets (on both nodes)

```bash
@strongswan1 @strongswan2 : PSK "yourStrongSharedSecret"
```

## System Settings

### Enable IP forwarding on both:
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
sysctl -w net.ipv4.ip_forward=1
```

Make it permanent:
```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
```

**Optional (if required)**
## iptables Rules

```bash
iptables -A FORWARD -s 192.168.10.0/24 -d 192.168.20.0/24 -j ACCEPT
iptables -A FORWARD -s 192.168.20.0/24 -d 192.168.10.0/24 -j ACCEPT
```


## Optional: Client2 route (if no default gateway)

```bash
ip route add 192.168.10.0/24 via 192.168.20.1
```

## Start StrongSwan and Verify

```bash
ipsec restart
ipsec statusall
```
## EXTRAs
## command to extract key
sudo ip  xfrm state
sudo ip xfrm policy

## How to add Route
 ```bash   
 sudo ip link set vEth0_0 up 
 ```
 #### kni1
 ```bash   
 sudo ip route add 172.16.10.0/24 dev vEth0_0
 ``` 
 (For @kni1)
#### kni2
```bash 
sudo ip route add 192.168.105.0/24 dev vEth0_0
``` 
(For @kni2)


## Test tunnel:
```bash
ping 192.168.20.2  # From Client1
ping 192.168.10.2  # From Client2
```

## Result

- Site-to-site tunnel established
- Clients can ping across securely over the IPsec tunnel
