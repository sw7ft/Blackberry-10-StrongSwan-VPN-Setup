# BB10 Firewall on ubuntu 20.04 LTS   

### Install StrongSwan

```
 sudo apt-get install strongswan
```

### configure the ipsec.conf file as follow,
 (replace leftip with your vpn server)

```
config setup
    charondebug="ike 1, cfg 2"

conn %default
    ikelifetime=60m
    keylife=20m
    rekeymargin=3m
    keyingtries=1
    keyexchange=ikev2


conn BB10
  rekey=no
   leftsubnet=0.0.0.0/0
   leftauth=psk
   leftid=54.39.41.29
   leftfirewall=yes
   right=%any
   rightsubnet=10.0.0.0/24
   rightsourceip=10.0.0.0/24
   rightdns=192.168.2.1
   rightauth=eap-mschapv2
   rightsendcert=never
   eap_identity=%any
   auto=add
```

### example ipsec.secrets file below

```
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.

: PSK "logitech"
matt : EAP "logitech"


```

### sysctl.conf edits

edit the following file located in /etc

```
sudo nano sysctl.conf

// Add the following lines to the file

net.ipv4.ip_forward=1
net.ipv4.conf.default.proxy_arp = 1
net.ipv4.conf.default.arp_accept = 1
net.ipv4.conf.default.proxy_arp_pvlan = 1

```


### IP Table Edits

make sure to change your ethernet adapter name by using ifconfig, mine is ens34

```
sudo iptables -A POSTROUTING -t nat -o ens34 -j MASQUERADE
sudo iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
sudo iptables -t nat -A POSTROUTING -s 10.0.0.1/24 -o ens34 -m policy --dir out
sudo iptables -t nat -A POSTROUTING -s 10.0.0.1/24 -o ens34 -j MASQUERADE
sudo iptables -A FORWARD -s 192.168.2.100/29 -j ACCEPT

sudo apt-get install iptables-persistent

```

### Restart Strong Swan

```
sudo ipsec restart

```

### Blackberry 10 Settings

the identity, username and password are in the ipsec.secrets file, you may have your own

```
- Add Vpn profile
- Enter VPN Server Address 
- Gateway Type: Generic IKEv2 VPN SERVER
- Authentication Type: EAP-MSCHAPv2
- Authentication ID Type: IPv4
- MSCHAPv2 EAP Identity: matt 
- MSCHAPv2 Username: matt
- MSCHAPv2 Password: logitech
- Gateway Auth Type: PSK
- Gateway Preshared Key: logitech

```