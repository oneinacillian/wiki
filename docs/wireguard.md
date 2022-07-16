> WireGuard is a secure network tunnel, operating at layer 3, implemented as a kernel virtual network
interface for Linux, which aims to replace both IPsec for most use cases, as well as popular user space and/or
TLS-based solutions like OpenVPN, while being more secure, more performant, and easier to use. It is essential to have traffic flowing between your hosts flowing via an encrypted tunnel.

### Install wireguard

**_NOTE:_** In this example, I will configure HOST-A on 172.168.25.1 and HOST-B on 172.168.25.2

On **HOST-A** install the wireguard suite as follows
```
sudo apt install wireguard
wg genkey > private
wg pubkey < private
ip link add wg0 type wireguard
ip addr add 172.168.25.1/24 dev wg0
wg set wg0 private-key ./private
ip link set wg0 up
wg showconf wg0 > /etc/wireguard/wg0.conf
echo "SaveConfig = true" >> /etc/wireguard/wg0.conf
wg-quick save wg0
wg show
```

Take note of interface: wg0 public key and listening port on HOST-A <br>
<img src="/assets/wg show.png"/>

Open up the firewall for the port on which wg expose the endpoint
```
firewall-cmd --add-port=<listening port>/tcp --permanent
service firewalld reload
```

On **HOST-B** configure as follows
```
sudo apt install wireguard
wg genkey > private
wg pubkey < private
ip link add wg0 type wireguard
ip addr add 172.168.25.2/24 dev wg0
wg set wg0 private-key ./private
ip link set wg0 up
wg showconf wg0 > /etc/wireguard/wg0.conf
echo "SaveConfig = true" >> /etc/wireguard/wg0.conf
wg-quick save wg0
```

Take note of interface: wg0 public key and listening port on HOST-B <br>
<img src="/assets/wg show.png"/>

```
firewall-cmd --add-port=<listening port>/tcp --permanent
service firewalld reload
```

**_NOTE:_** Having both HOST-A and HOST-B public key as well as port, you can now establish a vpn tunnel between the 2.<br>

On HOST-A execute the following =>
```
wg set wg0 peer <HOST-B public key> allowed-ips <172.168.25.2>/32 endpoint <public ip of HOST-B and port (ip:port)>
wg set wg0 peer <HOST-B public key> persistent-keepalive 25
wg-quick down wg0
wg-quick up wg0
```

On HOST-B execute the following =>
```
wg set wg0 peer <HOST-A public key> allowed-ips <172.168.25.1>/32 endpoint <public ip of HOST-A and port (ip:port)>
wg set wg0 peer <HOST-A public key> persistent-keepalive 25
wg-quick down wg0
wg-quick up wg0
```
The persistent-keepalive between HOST-A and HOST-B will send a packet to each peer to keep tunnel active
Both hosts should now be reachable via a secure VPN tunnel 