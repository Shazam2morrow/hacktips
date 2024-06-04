# Wireguard

```bash
$ sudo apt install wireguard
```

## Generate key pairs for the server

Generate private key for Wireguard and change its permissions:

```bash
$ wg genkey | sudo tee /etc/wireguard/private.key
$ sudo chmod go= /etc/wireguard/private.key
```

The `chmod go=` command removes any permissions on the file for users and groups other than the root user to ensure that only it can access the private key.

Create the corresponding public key, which is derived from the private key:

```bash
$ sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

If you are using WireGuard with IPv6, then you will need to generate a unique local IPv6 unicast address prefix based on the algorithm in RFC 4193. The addresses that you use with WireGuard will be associated with a virtual tunnel interface. You will need to complete a few steps to generate a random, unique IPv6 prefix within the reserved fd00::/8 block of private IPv6 addresses.

According to the RFC, the recommended way to obtain a unique IPv6 prefix is to combine the time of day with a unique identifying value from a system like a serial number or device ID. Those values are then hashed and truncated resulting in a set of bits that can be used as a unique address within the reserved private fd00::/8 block of IPs.

To get started generating an IPv6 range for your WireGuard Server, collect a 64-bit timestamp using the date utility with the following command:

```bash
$ date +%s%N

1714234091992817095
```

You will receive a number like the following, which is the number of seconds (the %s in the date command), and nanoseconds (the %N) since 1970-01-01 00:00:00 UTC combined together.

Record the value somewhere for use later in this section. Next, copy the machine-id value for your server from the /var/lib/dbus/machine-id file. This identifier is unique to your system and should not change for as long as the server exists:

```bash
$ cat /var/lib/dbus/machine-id 

ed3a87d8b5b59f62cf0e8c1d662d1cb0
```

Now you need to combine the timestamp with the machine-id and hash the resulting value using the SHA-1 algorithm. The command will use the following format:

```bash
$ printf <timestamp><machine-id> | sha1sum
```

```bash
$ printf DVd8b1knuuz/KfN8e7qSGSTSzeUcI8ARHOAzRp8p4Fc=ed3a87d8b5b59f62cf0e8c1d662d1cb0 | sha1sum

426647d0522c9f3f1b3908714e88a2ad22d6ef26  -
```

Note that the output of the sha1sum command is in hexadecimal, so the output uses two characters to represent a single byte of data. For example 4f and 26 in the example output are the first two bytes of the hashed data.

The algorithm in the RFC only requires the least significant (trailing) 40 bits, or 5 bytes, of the hashed output. Use the cut command to print the last 5 hexadecimal encoded bytes from the hash:

```bash
$ printf 426647d0522c9f3f1b3908714e88a2ad22d6ef26 | cut -c 31-

ad22d6ef26
```

The -c argument tells the cut command to select only a specified set of characters. The 31- argument tells cut to print all the characters from position 31 to the end of the input line.

Now you can construct your unique IPv6 network prefix by appending the 5 bytes you have generated with the fd prefix, separating every 2 bytes with a : colon for readability. Because each subnet in your unique prefix can hold a total of 18,446,744,073,709,551,616 possible IPv6 addresses, you can restrict the subnet to a standard size of /64 for simplicity.

Using the bytes previously generated with the /64 subnet size the resulting prefix will be the following:

```bash
Unique Local IPv6 Address Prefix

fdad:22d6:ef26::/64
```

This `fdad:22d6:ef26::/64` range is what you will use to assign individual IP addresses to your WireGuard tunnel interfaces on the server and peers. To allocate an IP for the server, add a `1` after the final `::` characters. The resulting address will be `fdad:22d6:ef26::1/64`. Peers can use any IP in the range, but typically you’ll increment the value by one each time you add a peer e.g. `fdad:22d6:ef26::2/64`.

Create a new configuration file using your preferred editor

```bash
$ sudo vim /etc/wireguard/wg0.conf

[Interface]
PrivateKey = CMx161/dQScJnCBymwcW7vgIIHH5twjRM72AUtdvVmY=
Address = 10.8.0.1/24, fdad:22d6:ef26::1/64
ListenPort = 80
SaveConfig = true
```
The `SaveConfig` line ensures that when a WireGuard interface is shutdown, any changes will get saved to the configuration file.

 If you would like to route your WireGuard Peer’s Internet traffic through the WireGuard Server then you will need to configure IP forwarding by following this section of the tutorial.

To configure forwarding, open the /etc/sysctl.conf file using nano or your preferred editor:

```bash
$ sudo vim /etc/sysctl.conf

# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

# Uncomment the next line to enable packet forwarding for IPv6
#  Enabling this option disables Stateless Address Autoconfiguration
#  based on Router Advertisements for this host
net.ipv6.conf.all.forwarding=1
```

To read the file and load the new values for your current terminal session, run:

```bash
$ sudo sysctl -p

net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

Now your WireGuard Server will be able to forward incoming traffic from the virtual VPN ethernet device to others on the server, and from there to the public Internet. Using this configuration will allow you to route all web traffic from your WireGuard Peer via your server’s IP address, and your client’s public IP address will be effectively hidden.

To allow WireGuard VPN traffic through the Server’s firewall, you’ll need to enable masquerading, which is an iptables concept that provides on-the-fly dynamic network address translation (NAT) to correctly route client connections.

First find the public network interface of your WireGuard Server using the ip route sub-command:
The public interface is the string found within this command’s output that follows the word “dev”. For example, this result shows the interface named eth0, which is highlighted below:

```bash
$ ip route list default

default via 167.172.32.1 dev eth0 proto static
```

To add firewall rules to your WireGuard Server, open the /etc/wireguard/wg0.conf file with nano or your preferred editor again.

```bash
$ sudo nano /etc/wireguard/wg0.conf

...
P# PostUp rules will run when the WireGuard server starts
# the virtual VPN tunnel.

# Allow forwarding IPv4 and IPv6 traffic that comes in on
# the wg0 VPN interface to the eth0 network interface on
# the server.
PostUp = ufw route allow in on wg0 out on eth0

# Masquerade and rewrite IPv4 traffic that comes in on the
# wg0 VPN interface to make it appear like it originates
# directly from the server's public IPv4 address.
PostUp = iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE

# Masquerade and rewrite IPv6 traffic that comes in on the
# wg0 VPN interface to make it appear like it originates
# directly from the server's public IPv6 address.
PostUp = ip6tables -t nat -I POSTROUTING -o eth0 -j MASQUERADE

# PreDown rules will run when the WireGuard server stops
# the virtual VPN tunnel.

# Undo the forwarding and masquerading rules for the VPN
# interface when the VPN is stopped.
PreDown = ufw route delete allow in on wg0 out on eth0
PreDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
PreDown = ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

```bash
$ sudo ufw allow 443/tcp
$ sudo ufw allow 80/udp
$ sudo ufw allow 53/udp
```

After adding those rules, disable and re-enable UFW to restart it and load the changes from all of the files you’ve modified:

```bash
$ sudo ufw disable
$ sudo ufw enable
```

You can confirm the rules are in place by running the ufw status command. Run it, and you should receive output like the following:

```bash
$ sudo ufw status

Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
80/udp                     ALLOW       Anywhere                  
53/udp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
80/udp (v6)                ALLOW       Anywhere (v6)             
53/udp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)     
```

WireGuard can be configured to run as a systemd service using its built-in wg-quick script. While you could manually use the wg command to create the tunnel every time you want to use the VPN, doing so is a manual process that becomes repetitive and error prone. Instead, you can use systemctl to manage the tunnel with the help of the wg-quick script.

Using a systemd service means that you can configure WireGuard to start up at boot so that you can connect to your VPN at any time as long as the server is running. To do this, enable the wg-quick service for the wg0 tunnel that you’ve defined by adding it to systemctl:

```bash
$ sudo systemctl -f enable wg-quick@wg0.service

Created symlink /etc/systemd/system/multi-user.target.wants/wg-quick@wg0.service → /lib/systemd/system/wg-quick@.service.
```

Notice that the command specifies the name of the tunnel wg0 device name as a part of the service name. This name maps to the /etc/wireguard/wg0.conf configuration file. This approach to naming means that you can create as many separate VPN tunnels as you would like using your server.

For example, you could have a tunnel device and name of prod and its configuration file would be /etc/wireguard/prod.conf. Each tunnel configuration can contain different IPv4, IPv6, and client firewall settings. In this way you can support multiple different peer connections, each with their own unique IP addresses and routing rules.

Now start the service:

```bash
$ sudo systemctl start wg-quick@wg0.service
```

Double check that the WireGuard service is active with the following command. You should see active (running) in the output:

```bash
$ sudo systemctl status wg-quick@wg0.service
```

The output shows the ip commands that are used to create the virtual wg0 device and assign it the IPv4 and IPv6 addresses that you added to the configuration file. You can use these rules to troubleshoot the tunnel, or with the wg command itself if you would like to try manually configuring the VPN interface.

Configuring a WireGuard peer is similar to setting up the WireGuard Server. Once you have the client software installed, you’ll generate a public and private key pair, decide on an IP address or addresses for the peer, define a configuration file for the peer, and then start the tunnel using the wg-quick script.

You can add as many peers as you like to your VPN by generating a key pair and configuration using the following steps. If you add multiple peers to the VPN be sure to keep track of their private IP addresses to prevent collisions.

```bash
$ sudo apt update
$ sudo apt install wireguard
```

Generate key pair on the client:

```bash
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

For remote peers that you access via SSH or some other protocol using a public IP address, you will need to add some extra rules to the peer’s wg0.conf file. These rules will ensure that you can still connect to the system from outside of the tunnel when it is connected. Otherwise, when the tunnel is established, all traffic that would normally be handled on the public network interface will not be routed correctly to bypass the wg0 tunnel interface, leading to an inaccessible remote system.

First, you’ll need to determine the IP address that the system uses as its default gateway. Run the following ip route command:

```bash
$ ip route list table main default

default via 172.31.255.1 dev wlp0s20f3 proto dhcp metric 600 
```

Note the gateway’s highlighted IP address `172.31.255.1` for later use, and device `wlp0s20f3`. Your device name may be different. If so, substitute it in place of `wlp0s20f3` in the following commands.

Next find the public IP for the system by examining the device with the ip address show command:

```bash
$ ip -brief address show wlp0s20f3

wlp0s20f3        UP             172.31.255.231/24 fe80::f62:947c:7fd1:5081/64
```

In this example output, the highlighted `172.31.255.231` IP (without the trailing /20) is the public address that is assigned to the `wlp0s20f3` device that you’ll need to add to the WireGuard configuration.

```bash
$ sudo vim /etc/wireguard/wg0.conf

# PostUp rules will run when the WireGuard server starts
# the virtual VPN tunnel.

# Create a custom routing rules to ensure that public
# traffic to the system uses the default gateway.

# Create a rule that checks for any routing entries in the table
# numbered 200 when the IP matches the system's public address.
PostUp = ip rule add table 200 from 172.31.255.231

# Create a rule that ensures that any traffic being processed
# by the 200 table will use the gateway routing, instead of the
# WireGuard interface.
PostUp = ip route add table 200 default via 172.31.255.1

# PreDown rules will run when the WireGuard server stops
# the virtual VPN tunnel.

# Undo the custom rules and routes when the tunnel is shutdown.
PreDown = ip rule delete table 200 from 172.31.255.231
PreDown = ip route delete table 200 default via 172.31.255.1
```
The table number 200 is arbitrary when constructing these rules. You can use a value between 2 and 252, or you can use a custom name by adding a label to the /etc/iproute2/rt_tables file and then referring to the name instead of the numeric value.

If you are using the WireGuard Server as a VPN gateway for all your peer’s traffic, you will need to add a line to the [Interface] section that specifies DNS resolvers. If you do not add this setting, then your DNS requests may not be secured by the VPN, or they might be revealed to your Internet Service Provider or other third parties.

To add DNS resolvers to your peer’s configuration, first determine which DNS servers your WireGuard Server is using. Run the following command on the WireGuard Server, substituting in your ethernet device name in place of eth0 if it is different from this example:

```bash
$ resolvectl dns eth0

Link 2 (eth0): 67.207.67.3 67.207.67.2
```

Before connecting the peer to the server, it is important to add the peer’s public key to the WireGuard Server. This step ensures that you will be able to connect to and route traffic over the VPN. Without completing this step the WireGuard server will not allow the peer to send or receive any traffic over the tunnel.

Ensure that you have a copy of the base64 encoded public key for the WireGuard Peer by running:

Now log into the WireGuard server, and run the following command:

```bash
$ sudo wg set wg0 peer YqtyhVt2cg0LmzRteexkVAfMaFPliB56dICxhGG1p2g= allowed-ips 10.8.0.2,fdad:22d6:ef26::2
```

Note that the allowed-ips portion of the command takes a comma separated list of IPv4 and IPv6 addresses. You can specify individual IPs if you would like to restrict the IP address that a peer can assign itself, or a range like in the example if your peers can use any IP address in the VPN range. Also note that no two peers can have the same allowed-ips setting.

Once you have run the command to add the peer, check the status of the tunnel on the server using the wg command:

```bash
$ sudo wg

interface: wg0
  public key: DVd8b1knuuz/KfN8e7qSGSTSzeUcI8ARHOAzRp8p4Fc=
  private key: (hidden)
  listening port: 80

peer: YqtyhVt2cg0LmzRteexkVAfMaFPliB56dICxhGG1p2g=
  allowed ips: 10.8.0.2/32, fdad:22d6:ef26::2/128
```

peer private key: wCvfnXzPvJUjLRq/9bBGGxc9jcHPswLSzQGSRZl/gEM=
peer public key: YqtyhVt2cg0LmzRteexkVAfMaFPliB56dICxhGG1p2g=