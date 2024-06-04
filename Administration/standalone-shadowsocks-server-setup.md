# How To Configure Standalone Shadowsocks Server

This guide will walk you through a set up process of standalone **Shadowsocks** server supporting multiple clients.

## Prerequisites

A server (virtual or physical) with at least **1 CPU**, **1024 MB RAM**, **25 GB free HDD or SDD**, running **Ubuntu 22.04**.

## Tested clients

### Desktop clients

- **Windows 11 Pro 23H2** using **Shadowsocks** version **4.4.1.0**.
- **Ubuntu 22.04.4** using **shadowsocks-libev** version **3.3.5**.
- **macOS Big Sur 11.7.10** using **ShadowsocksX-NG** version **1.10.2**.

### Mobile clients

- **Android 14** using **Shadowsocks** version **5.2.6**.
- **iOS 16.6.1** using **Shadowrocket** version **2.2.46**.

## Introduction

Shadowsocks is a secure, open-source, and high-performance **SOCKS5** proxy designed to protect your internet traffic.

It was developed as a response to increasing internet censorship, particularly the **Great Firewall of China**. The goal was to provide a method for users to bypass internet restrictions and access blocked content securely and privately.


It uses **SOCKS5** proxy, which can handle any kind of network protocol (like HTTP, FTP, SMTP, etc.), making it versatile. The encryption between the client and server ensures that the data cannot be easily intercepted or read, providing a layer of security and privacy for users.

## How it works

**Shadowsocks** allows for secure and encrypted connections between the client and the server, effectively bypassing internet censorship and ensuring privacy.

The **Shadowsocks** network consists of multiple components, including a local component (**ss-local**) that acts like a traditional **SOCKS5** server, and a remote component (**ss-remote**) that decrypts and forwards data to the target. The communication between these components is encrypted to ensure data security:

`client <-> ss-local <-[encrypted-traffic]-> ss-remote <-> target`

Below you will find information on how to configure your own **Shadowsocks** server.

## Server configuration

### Step 1 - Updating system packages

Before we proceed with configuration we have to make sure we have the latest updates for our system (especially the security ones):

To update your system run the following commands in a sequence:

```bash
$ sudo apt update
$ sudo apt upgrade
$ sudo reboot
```
After that your server will be restarted and you will have to reconnect to it after a few minutes.

### Step 2 - Installing shadowsocks-libev

There are multiple implementations of **Shadowsocks** written on Python, C, Go and Rust.

`shadowsocks-libev` is a lightweight C implementation for embedded devices and low end boxes. It has a very small footprint (several megabytes) for thousands of connections. I will use this implementation as a reference throughout the tutorial.

It is available in the official repository for Ubuntu and you can install one using the following command:

```bash
$ sudo apt install shadowsocks-libev
```

### Step 3 - Create configuration file

The default configuration file is located at `/etc/shadowsocks-libev/config.json`.

Open it using any text editor:

```bash
$ sudo vim /etc/shadowsocks-libev/config.json
```

Inside you can see an example of configuration:

```bash
{
    "server":["::1", "127.0.0.1"],
    "mode":"tcp_and_udp",
    "server_port":8388,
    "local_port":1080,
    "password":"WlptXpclLjc6",
    "timeout":86400,
    "method":"chacha20-ietf-poly1305"
}
```

We will create our own configuration so we do not need this one and you can remove everything inside the file and paste the following lines instead:

```bash
{
    "server": "[:SERVER_IP:]",
    "server_port": 443,
    "password": "[:PASSWORD:]",
    "timeout": 10,
    "method": "aes-256-gcm",
    "fast_open": true,
    "nameserver": "1.1.1.1",
    "mode": "tcp_and_udp"
}
```

Let's break down each line:
- `server` allows you to specify the server's public IP address or hostname. You need to change `[:SERVER_IP:]` to the public IP address of your server. You can find it in the server's management console or using `ifconfig` command. Clients will use this IP address to connect to your server.
- `server_port` allows you to specify the port on which our proxy will listen for incoming connections. I recommend to use `443` or `80` port because they are used for **HTTP(S)** connections and are harder to block by providers.
- `password` allows you to specify the password that both the server and the client know to communicate with each other. I also recommend to create a very strong password (at least 20 characters) and replace `[:PASSWORD:]` with it. Keep it secure because everyone who knows it will be able to access your server.
- `timeout` allows you to specify the socket timeout in seconds. If no connection is established during this timeframe then the connection attempt is considered as failed.
- `method` allows you to specify the cipher suite that will be used for encrypting all messages between the server and the client. `aes-256-gcm` is the recommended algorithm because it provides good speed and strong confidentiality.
- `fast_open` allows you to open TCP connections as fast as possible which is good for performance.
- `nameserver` allows you to specify custom DNS resolvers. I recommend to use Cloudflare `1.1.1.1` but you can use any other resolver.
- `mode` allows you to specify in which mode the server will operate. `tcp_and_udp` means that the server will support both **TCP** and **UDP**.

So far we have created our custom configuration and we have everything we need to start the proxy server but before we do that we need to add some additional configuration directives to our system.

### Step 4 - Linux server optimizations

The steps below allow you to fine-tune your **Shadowsocks** server to increase its performance.

#### Increase the maximum number of open file descriptors 

To handle thousands of concurrent **TCP** connections, we should increase the limit of file descriptors opened.

Open `/etc/security/limits.conf` file using any text editor and add the following lines:

```bash
$ sudo vim /etc/security/limits.conf

* soft nofile 51200
* hard nofile 51200
```

Then, before you start the proxy server, set the `ulimit` first using the command below:

```bash
$ ulimit -n 51200
```

#### Tune the kernel parameters 

The principles of tuning parameters for **Shadowsocks** are:

1. Reuse ports and connections as soon as possible.
2. Enlarge the queues and buffers as large as possible.
3. Choose the **TCP** congestion algorithm for large latency and high throughput.

In order to do it you need to edit `/etc/sysctl.conf` file using any text editor:

```bash
$ sudo vim /etc/sysctl.conf
```

And paste the following lines at the end of the file:

```bash
fs.file-max = 51200

net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096

net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mem = 25600 51200 102400
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
```

Now you need to reload the configuration file at runtime using the following command:

```bash
$ sudo sysctl -p
```

Otherwise new configuration will be applied after the server reboots.

### Step 5 - Configure systemd management 

To manage **Shadowsocks** service we will use `systemd` which is the preferred way to manage services in modern Ubuntu distros.

You can find the default `systemd` file for `shadowsocks-libev` service here `/lib/systemd/system/shadowsocks-libev.service`.

You can view its content using any text editor:

```bash
$ sudo vim /lib/systemd/system/shadowsocks-libev.service
```

The content of the file will look like this:

```bash
#  This file is part of shadowsocks-libev.
#
#  Shadowsocks-libev is free software; you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 3 of the License, or
#  (at your option) any later version.
#
#  This file is default for Debian packaging. See also
#  /etc/default/shadowsocks-libev for environment variables.

[Unit]
Description=Shadowsocks-libev Default Server Service
Documentation=man:shadowsocks-libev(8)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
DynamicUser=true
EnvironmentFile=/etc/default/shadowsocks-libev
LimitNOFILE=32768
ExecStart=/usr/bin/ss-server -c $CONFFILE $DAEMON_ARGS

[Install]
WantedBy=multi-user.target
```

In case you need to change how the `systemd` manages `shadowsocks-libev` service you can edit this file directly.

Notice that this service uses `DynamicUser` option which enables dynamic user creation specifically for this service and after it is stopped the user will be removed.

This file uses environment variables from `/etc/default/shadowsocks-libev` that is used to pass different arguments to the `ss-server` command (it is the command that is used to start **Shadowsocks** server).

You can view it using any text editor:

```bash
$ sudo /etc/default/shadowsocks-libev
```

The content of the file will look like this:

```bash
# Defaults for shadowsocks initscript
# sourced by /etc/init.d/shadowsocks-libev
# installed at /etc/default/shadowsocks-libev by the maintainer scripts

#
# This is a POSIX shell fragment
#
# Note: `START', `GROUP' and `MAXFD' options are not recognized by systemd.
# Please change those settings in the corresponding systemd unit file.

# Configuration file
CONFFILE="/etc/shadowsocks-libev/config.json"

# Extra command line arguments
DAEMON_ARGS=

# User and group to run the server as
USER=nobody
GROUP=nogroup

# Number of maximum file descriptors
MAXFD=32768
```

In case you need to pass additional arguments to the `ss-server` command you can define them here as well as other variables.

You do not need to change any of these files until you want to change the default behavior of **Shadowsocks** service which is sufficient in most cases.

### Step 6 - Enabling service

Now we need to enable `shadowsocks-libev` service to run even after our server reboots and finally start the proxy server itself using the command below:

```bash
$ sudo systemctl enable shadowsocks-libev.service
$ sudo systemctl start shadowsocks-libev.service
```

In order to view the status of the service run the command below:

```bash
$ sudo systemctl status shadowsocks-libev.service
```

The output will look like this:

```bash
shadowsocks-libev.service - Shadowsocks-libev Default Server Service
     Loaded: loaded (/lib/systemd/system/shadowsocks-libev.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2024-04-28 23:47:39 UTC; 24min ago
       Docs: man:shadowsocks-libev(8)
   Main PID: 8792 (ss-server)
      Tasks: 1 (limit: 1101)
     Memory: 6.5M
        CPU: 4.257s
     CGroup: /system.slice/shadowsocks-libev.service
             └─8792 /usr/bin/ss-server -c /etc/shadowsocks-libev/config.json
```

If the service is active and enabled then you configured your **Shadowsocks** server correctly and it is time to configure a firewall.

### Step 7 - Firewall configuration

In order to allow client to connect to our server we need to add appropriate rules to our firewall.

Ubuntu has `ufw` tool that allows you to manage firewall rules very easily.

The following command allows anyone to connect to the server's port **443** using either **TCP** or **UDP**:

```bash
$ sudo ufw allow 443/tcp
$ sudo ufw allow 443/udp
```

Now you need to restart the `ufw` using commands below:

```bash
$ sudo ufw disable
$ sudo ufw enable
````

To view status of the firewall run this command:

```bash
$ sudo ufw status
```

The output should look like this:

```bash
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
443/udp                    ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
443/udp (v6)               ALLOW       Anywhere (v6)    
```

Now clients can connect to our server.

## Client configuration

### Desktop clients

#### Windows

Install the latest release of **Shadowsocks** client from the [GitHub repository](https://github.com/shadowsocks/shadowsocks-windows/releases).

Extract the content of the archive and run the client.

You need to fill all fields using information from the provided configuration file. After filling out all form fields save it and press **OK**.

Now you can open your browser and change proxy settings to point to `127.0.0.1:1080`. Ensure that you selected an option to proxy your DNS requests too. If you do not do that then your **DNS** requests will be visible to your provider.

As the last step check your device for **DNS** leakage by following the instructions in the [DNS Leak](#check-for-dns-leaks) section.

#### MacOS

Install the latest release of **ShadowsocksX-NG** client from the [GitHub repository](https://github.com/shadowsocks/ShadowsocksX-NG/releases/).

Import the installation image and follow installation instructions and after that run the client.

You need to fill all fields using information from the provided configuration file. After filling out all form fields save it and press **OK**.

Now you can open your browser and change proxy settings to point to `127.0.0.1:1080`. Ensure that you selected an option to proxy your DNS requests too. If you do not do that then your **DNS** requests will be visible to your provider.

As the last step check your device for **DNS** leakage by following the instructions in the [DNS Leak](#check-for-dns-leaks) section.

#### Linux

Install the `shadowsocks-libev` package from the default Ubuntu repository using the command below:

```bash
sudo apt install shadowsocks-libev
```

Now stop and disable the `shadowsocks-libev` service using the following commands:

```bash
$ sudo systemctl stop shadowsocks-libev
$ sudo systemctl disable shadowsocks-libev
```

We need to create local configuration file for our client at `/etc/shadowsocks-libev/local-config.json` using any text editor:

```bash
$ sudo vim /etc/shadowsocks-libev/local-config.json
```

Paste the following lines to the file:

```bash
{
    "server": "[:SERVER_IP:]",
    "mode": "tcp_and_udp",
    "server_port": 443,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "[:PASSWORD:]",
    "timeout": 10,        
    "method": "aes-256-gcm"
}
```

The client configuration file contains the same parameters as the server one except a few:
- `local_address` enables you to specify the local address to use while the client is making outbound connections to the server.
- `local_port` enables you to specify which local port to use while the client is making outbound connections to the server.

The next step is to start and enable our proxy client using the following commands: 

```bash
$ sudo systemctl start shadowsocks-libev-local@local-config.service
$ sudo systemctl enable shadowsocks-libev-local@local-config.service
```

Now you need to check the status of the service using the command below:

```bash
$ sudo systemctl status shadowsocks-libev-local@local-config.service
```

The output will look like this:

```bash
shadowsocks-libev-local@local-config.service - Shadowsocks-Libev Custom Client Service for local/config
     Loaded: loaded (/lib/systemd/system/shadowsocks-libev-local@.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-04-29 23:45:30 +05; 7min ago
       Docs: man:ss-local(1)
   Main PID: 13939 (ss-local)
      Tasks: 1 (limit: 18661)
     Memory: 1.0M
        CPU: 64ms
     CGroup: /system.slice/system-shadowsocks\x2dlibev\x2dlocal.slice/shadowsocks-libev-local@local-config.service
             └─13939 /usr/bin/ss-local -c /etc/shadowsocks-libev/local-config.json
...
```

If the service is active then you have configured everything correctly.

Now you can open your browser and change proxy settings to point to `127.0.0.1:1080`. Ensure that you selected an option to proxy your DNS requests too. If you do not do that then your DNS requests will be visible to your provider.

As the last step check your device for **DNS** leakage by following the instructions in the [DNS Leak](#check-for-dns-leaks) section.

### Mobile clients

#### Android

To connect to **Shadowsocks** server from your **Android** device you need to install [Shadowsocks for Android](https://github.com/shadowsocks/shadowsocks-android).

After installation open the app and tap on add button (**plus sign**) and import configuration using any convenient way, for example, by scanning a **QR** code.

After the successful import tap on edit button (**pencil**) and make sure that the following options are set:
- "**Routes**: **All**".
- "**VPN mode for selected apps**: **Disabled**": if you need to proxy traffic only for specific applications you can enable this option and choose the desired apps.

Now tap on the selected profile and then on connect button (**paper plane**) and allow a VPN connection by pressing **OK**.

Now you need to check your connection by tapping on the area under the connect button. It should say something like "**Success: HTTPS handshake took x ms**". If you see this message then the connection between the client and the server was established successfully.

As the last step check your device for **DNS** leakage by following the instructions in the [DNS Leak](#check-for-dns-leaks) section.

#### IOS

To connect to **Shadowsocks** server from your **iOS** device you need to install [Shadowrocket](https://apps.apple.com/us/app/shadowrocket/id932747118) app. This application is not-free and you have to pay in order to use it. You can also use another client if you want but ensure that it supports **Shadowsocks** protocol.

After you install the app you need to open it and set "Global Routing" to "Proxy". This setting will enforce to send all traffic from your device through the **Shadowsocks** server.

Press the plus sign and fill the form with your data and save it. You must provide the following details:
- **Address**: public IP address of your server.
- **Port**: the port of your server.
- **Method**: encryption algorithm.
- **Password**: password for your proxy.

Now select your proxy and enable it in the upper part of the screen.

As the last step check your device for **DNS** leakage by following the instructions in the [DNS Leak](#check-for-dns-leaks) section.

## How to generate a QR code

**Shadowsocks** for **Android** and **iOS** also accepts **BASE64** encoded **URI** format configs:

```
ss://BASE64-ENCODED-STRING-WITHOUT-PADDING#TAG
```

Where the plain **URI** should be:

```
ss://method:password@hostname:port
```

Note that the above **URI** does not follow **RFC3986**. It means the password here should be plain text, not percent-encoded.

For example, we have a server at `192.168.100.1:8888` using `bf-cfb` encryption method and password `test/!@#:`. Then, with the plain **URI** `ss://bf-cfb:test/!@#:@192.168.100.1:8888`, we can generate the **BASE64** encoded **URI**:

```javascript
> console.log("ss://" + btoa("bf-cfb:test/!@#:@192.168.100.1:8888"))
ss://YmYtY2ZiOnRlc3QvIUAjOkAxOTIuMTY4LjEwMC4xOjg4ODg
```

To help organize and identify these **URIs**, you can append a tag after the **BASE64** encoded string:

```
ss://YmYtY2ZiOnRlc3QvIUAjOkAxOTIuMTY4LjEwMC4xOjg4ODg#example-server
```

This **URI** can also be encoded to **QR** code, for example, like this:

```bash
echo "ss://YmYtY2ZiOnRlc3QvIUAjOkAxOTIuMTY4LjEwMC4xOjg4ODg#example-server" | qrencode -o shadowsocks-conf.png
```

Then, you just need to scan it with your **Android** or **iOS** device.

## Check for DNS leaks

After you have successfully connected to the **Shadowsocks** server you need to check that your device does not leak any **DNS** requests to your provider.

To check it you need to do the following steps:
1. Open a browser on your device on go to [https://www.dnsleaktest.com/](https://www.dnsleaktest.com/).
2. When the website opens click on **Extended test** button and wait until all checks are completed.
3. After all tests are completed ensure that you do not see any records about the country where you are physically located now.
4. If you do not see such records then you configured everything correctly and you can continue to use this proxy.
5. If you see that some records contain information about country where you are currently in then you might have configured something incorrectly. Check settings of the client app you are currently using. Maybe you need to enable or disable certain parameters to make it work.

## References

- [Shadowsocks - A Fast Tunnel Proxy That Helps You Bypass Firewalls](https://shadowsocks.org/)
- [shadowsock-libev - Source Code](https://github.com/shadowsocks/shadowsocks-libev)
- [Shadowsocks - Documentation](https://shadowsocks.org/doc/what-is-shadowsocks.html)