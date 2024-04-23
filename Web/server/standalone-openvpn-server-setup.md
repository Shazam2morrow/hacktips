# How To Configure Standalone OpenVPN Server

This guide will work you through a set up process of standalone OpenVPN server supporting multiple clients.

## Prerequisites

### Minimum server requirements

Server with **1 CPU**, **1024 MB RAM**, **25 GB free HDD or SDD**, running **Ubuntu 22.04**.

### Clients tested

- **Ubuntu 22.04.4** using openvpn version `2.5.9`.
- **Android 14** using OpenVPN Connect version `3.4.1`. 

### Hosts

In order to follow this tutorial you will need two Ubuntu hosts with a sudo non-root and a firewall enabled on each of them:
- The first server will be used as a VPN server which we will refer to as **OpenVPN Server**.
- The second server will be used as a private **certificate authority (CA)** server which we will refer to as **CA Server**.

In addition to that, you will need a client machine which you will use to connect to your **OpenVPN Server**.

### Wny should I use standalone CA server?

For optimal security, it's advised not to use your **OpenVPN Server** or local machine as your CA. Doing so could expose your VPN to potential vulnerabilities. [According to the official OpenVPN documentation](https://openvpn.net/community-resources/how-to/#setting-up-your-own-certificate-authority-ca-and-generating-certificates-and-keys-for-an-openvpn-server-and-multiple-clients), the best practice is to set up your CA on a standalone machine dedicated to handling certificate requests.

## Introduction

OpenVPN is a versatile SSL VPN solution that implements secure network extension at OSI layer 2 or 3 using the industry-standard SSL/TLS protocol. It supports various client authentication methods including certificates, smart cards, or username/password credentials. 

Additionally, OpenVPN allows for user or group-specific access control policies through firewall rules applied to the VPN virtual interface. It's important to note that **OpenVPN is not a web application proxy and does not operate through a web browser**.

It is also important to note that if you are renting a server from one of the popular cloud providers you must be aware that **they usually charge for bandwidth overages**. So be mindful of how much traffic your server is handling.

To follow the rest of the tutorial you must be able to log in to your servers using SSH as sudo non-root user. During this guide user `johhdoe` will have such privileges.

## Step 1 - Updating system packages

Before we proceed with configuration we have to make sure we have the latest updates for our system (especially the security ones):

To update your system run the following commands in a sequence:

```bash
$ sudo apt update
$ sudo apt upgrade
$ sudo reboot
```
After that your server will be restarted and you will have to reconnect to it after a few minutes.

## Step 2 - Installing OpenVPN and Easy-RSA

In this step we will install OpenVPN and Easy-RSA from the default Ubuntu repositories.

Easy-RSA is a **public key infrastructure (PKI)** management tool that you will use on the **OpenVPN Server** to generate a certificate request that you will then verify and sign on the **CA Server**.

Run the command below to install them:

```bash
$ sudo apt install openvpn easy-rsa
```

Next you will need to create a new directory on the **OpenVPN Server** as your non-root user called `~/easy-rsa`:

```bash
$ mkdir ~/easy-rsa
```

Now you will need to create a symlink from the `easyrsa` script that the package installed into the `~/easy-rsa` directory that you just created:

```bash
$ ln -s /usr/share/easy-rsa/* ~/easy-rsa/
```

As a result, any updates to the `easy-rsa` package will be automatically reflected in your PKI’s scripts.

Finally, ensure the directory’s owner is your non-root sudo user and restrict access to that user using `chmod`:

```bash
$ chown johndoe ~/easy-rsa
$ chmod 700 /home/johndoe/easy-rsa
```

## Step 3 - Creating a PKI for OpenVPN

Before generating your OpenVPN server’s private key and certificate, it’s crucial to establish a local PKI directory on your server. This directory serves as the central hub for managing certificate requests from both the server and clients, rather than generating them directly on your **CA Server**.

In the `vars` file, you will input default values that will be used throughout the certificate creation process. This step ensures consistency and ease of management when handling certificate requests:

```bash
$ cd ~/easy-rsa
$ vim vars
```

Once the file is opened, paste in the following two lines:

```bash
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
```

Configuring your OpenVPN and CA servers to utilize **Elliptic Curve Cryptography (ECC)** allows for faster key exchanges between clients and the server. This is achieved through **Elliptic Curve** algorithms, which are notably quicker than the traditional **RSA** algorithm used with **Diffie-Hellman** for key exchanges.

When clients connect to the **OpenVPN Server**, they initiate a TLS handshake using asymmetric encryption. However, once the connection is established, symmetric encryption, aka **shared key encryption**, is used for transmitting VPN traffic.

Symmetric encryption offers reduced computational overhead compared to asymmetric encryption due to smaller numbers and optimized operations on modern CPUs. The transition from asymmetric to symmetric encryption involves utilizing the **Elliptic Curve Diffie-Hellman (ECDH)** algorithm to swiftly agree upon a shared secret key between the **OpenVPN Server** and client.

Once the `vars` file is configured, proceed to create the PKI directory by executing the `easyrsa` script with the `init-pki` option. Despite running this command previously on the **CA Server**, it's essential to repeat it here as the **OpenVPN Server** and **CA Server** maintain separate PKI directories:

```bash
$ ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/johndoe/easy-rsa/pki
```

Remember, the **OpenVPN Server** does not function as a CA; its PKI directory serves solely for storing certificate requests and public certificates.

## Step 4 — Creating an **OpenVPN Server** certificate request and private key

Once all prerequisites are installed on your **OpenVPN Server**, the next step is generating a private key and **Certificate Signing Request (CSR)**. This can be done by navigating to the `~/easy-rsa` directory on your server as your non-root user.

After generating the CSR, transfer it to your **CA Server** for signing, thereby creating the necessary certificate. Once the certificate is signed, transfer it back to the **OpenVPN Server** and install it for the server's use.

To generate a certificate request for your machine, you will use `easyrsa` with the `gen-req` option. It's important to specify a **Common Name (CN)** for the machine. This CN can be any descriptive term you choose.

When using `easyrsa`, ensure to include the `nopass` option. This prevents the request file from being password-protected, which could potentially cause permissions issues down the line:

```bash
$ ./easyrsa gen-req openvpn nopass

...

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [openvpn]:

Keypair and certificate request completed. Your files are:
req: /home/johndoe/easy-rsa/pki/reqs/openvpn.req
key: /home/johndoe/easy-rsa/pki/private/openvpn.key
```

In this case CN of our certificate is `openvpn`.

This will create a private key for the server and a certificate request file called `openvpn.req`. Copy the server key to the `/etc/openvpn/server` directory:

```bash
$ sudo cp /home/johndoe/easy-rsa/pki/private/openvpn.key /etc/openvpn/server/
```

Once you have completed these steps, you have successfully generated a private key for your **OpenVPN Server** and created a CSR. The CSR is now ready for signing by your **CA Server**'s private key.

## Step 5 — Signing the OpenVPN server’s certificate request

After generating a CSR and a private key for your **OpenVPN Server**, the next step involves informing the **CA Server** about the server certificate and validating it. Once the CA has validated the certificate and sent it back to the **OpenVPN Server**, clients who trust your CA will also trust the **OpenVPN Server**.

You can copy the `openvpn.req` certificate request to the **CA Server** for signing using any preferable way but I recommend to use SCP utility. It uses SSH connection to securely connect files to and from the server.

You can copy files directly from your **OpenVPN Server** to **CA Server** an vise versa but in order to do it you have to configure SSH keys on both ends.

Another way is to use your local machine to first download a file from one server and then send it to another server.

The command below will download `openvpn.req` from the **OpenVPN Server** and store it at the `/home/johndoe/Downloads` folder on the local machine:

```bash
$ scp johndoe@[OPENVPN_SERVER_IP]:/home/johndoe/easy-rsa/pki/reqs/openvpn.req ~/Downloads/openvpn.req
```

The command below will upload file at `/home/johndoe/Downloads/openvpn.req` on the local host at `/tmp` folder on the **CA Server**:

```bash
$ scp ~/Downloads/openvpn.req johndoe@[CA_SERVER_IP]:/tmp
```

The next step is to log in to the **CA Server** as the non-root user that you created to manage your CA. You will `cd` to the `~/easy-rsa` directory where you created your PK and then import the certificate request using the `easyrsa` script:

```bash
$ cd ~/easy-rsa
$ ./easyrsa import-req /tmp/openvpn.req openvpn

...

The request has been successfully imported with a short name of: openvpn
You may now use this name to perform signing operations on this request.
```

To proceed, sign the request by executing the `easyrsa` script, utilizing the `sign-req` option along with the request type and the CN. The request type, either `client` or `server`, determines the signing process. Since you are dealing with the OpenVPN Server’s certificate request, ensure to specify the server request type.

```bash
$ ./easyrsa sign-req server openvpn

...

Write out database with 1 new entries
Data Base Updated

Certificate created at: /home/johndoe/easy-rsa/pki/issued/openvpn.crt
```

After encrypting your CA private key, remember that you will need to enter your password when prompted.

Once you have completed these steps, you have effectively signed the **OpenVPN Server**'s certificate request using the **CA Server**'s private key. The resulting `openvpn.crt` file includes the **OpenVPN Server**'s public encryption key along with a signature from the **CA Server**. This signature serves as assurance to anyone who trusts the **CA Server** that they can trust the **OpenVPN Server** when connecting to it.

To finish configuring the certificates, copy the `openvpn.crt` and `ca.crt` files from the **CA Server** to the **OpenVPN Server** using your local host:

```bash
$ scp johndoe@[CA_SERVER_IP]:/home/johndoe/easy-rsa/pki/issued/openvpn.crt ~/Downloads/openvpn.crt

$ scp johndoe@[CA_SERVER_IP]:/home/johndoe/easy-rsa/pki/ca.crt ~/Downloads/ca.crt

$ scp ~/Downloads/openvpn.crt johndoe]@[OPENVPN_SERVER_IP]:/tmp

$ scp ~/Downloads/ca.crt johndoe]@[OPENVPN_SERVER_IP]:/tmp
```

Now back on your **OpenVPN Server**, copy the files from `/tmp` to /`etc/openvpn/server`:

```bash
$ sudo cp /tmp/{server.crt,ca.crt} /etc/openvpn/server
```

## Step 6 - Configuring OpenVPN cryptographic material

To enhance security, incorporate an additional shared secret key for both the server and clients, employing OpenVPN’s `tls-crypt` directive. This feature serves to obscure the TLS certificate utilized during the initial connection between server and client.

Additionally, it facilitates rapid verification of incoming packets by the **OpenVPN Server**: packets signed with the pre-shared key are processed, while those lacking a signature are recognized as originating from an untrusted source and can be promptly discarded, eliminating the need for further decryption.

By implementing this option, your **OpenVPN Server** gains resilience against unauthenticated traffic, port scans, and Denial of Service attacks, thereby safeguarding server resources. Moreover, it increases the complexity of identifying OpenVPN network traffic.

To generate the `tls-crypt` pre-shared key, run the following on the **OpenVPN Server** in the `~/easy-rsa` directory:

```bash
$ cd ~/easy-rsa
$ openvpn --genkey secret ta.key
```

The result will be a file called `ta.key`. Copy it to the `/etc/openvpn/server/` directory:

```bash
$ sudo cp ta.key /etc/openvpn/server
```

With these files in place on the **OpenVPN Server** you are ready to create client certificates and key files for your users, which you will use to connect to the VPN.

## Step 7 — Generating a client certificate and key pair

This section outlines a streamlined process for generating a certificate request directly on the **OpenVPN Server**, eliminating the need to transfer keys, certificates, and configuration files to clients manually. By following this approach, you can automate the creation of client configuration files, simplifying the process of joining the VPN.

You will generate a single client key and certificate pair following the steps outlined in this guide. If you have multiple clients, you can replicate this process for each one, ensuring to assign a unique name value to the script for every client. Throughout this tutorial, the initial certificate/key pair is denoted as `e8b4e806-ac48-4cd5-b7e6-4e714711cc50`.

Get started by creating a directory structure within your home directory to store the client certificate and key files:

```bash
$ mkdir -p ~/client-configs/keys
```

Since you will store your clients’ certificate/key pairs and configuration files in this directory, you should lock down its permissions now as a security measure:

```bash
$ chmod -R 700 ~/client-configs
```

While you can choose any CN for your client's key I suggest to use something unique like **Universally Unique Identifier (UUID)**. You can generate one by running the following command:

```bash
$ uuidgen

e8b4e806-ac48-4cd5-b7e6-4e714711cc50
```

Next, navigate back to the EasyRSA directory and run the `easyrsa` script with the `gen-req` and `nopass` options, along with the CN for the client:

```bash
$ cd ~/easy-rsa
$ ./easyrsa gen-req e8b4e806-ac48-4cd5-b7e6-4e714711cc50 nopass
```

Press `ENTER` to confirm the common name. Then, copy the `e8b4e806-ac48-4cd5-b7e6-4e714711cc50.key` file to the `~/client-configs/keys/` directory you created earlier:

```bash
$ cp pki/private/e8b4e806-ac48-4cd5-b7e6-4e714711cc50.key ~/client-configs/keys/
```

Next, transfer the `e8b4e806-ac48-4cd5-b7e6-4e714711cc50.req` file to your **CA Server** using a secure method:

```bash
$ scp pki/reqs/e8b4e806-ac48-4cd5-b7e6-4e714711cc50.req johndoe@[CA_SERVER_IP]:/tmp
```

Now log in to your **CA Server**. Then, navigate to the EasyRSA directory, and import the certificate request:

```bash
$ cd ~/easy-rsa
$ ./easyrsa import-req /tmp/e8b4e806-ac48-4cd5-b7e6-4e714711cc50.req e8b4e806-ac48-4cd5-b7e6-4e714711cc50
```

Next, sign the request the same way as you did for the server in the previous step. This time, though, be sure to specify the client request type:

```bash
$ ./easyrsa sign-req client e8b4e806-ac48-4cd5-b7e6-4e714711cc50

Type the word 'yes' to continue, or any other input to abort.
Confirm request details: yes
```

When prompted, enter `yes` to confirm that you intend to sign the certificate request and that it came from a trusted source.

Again, if you encrypted your CA key, you’ll be prompted for your password here.

This will create a client certificate file named `e8b4e806-ac48-4cd5-b7e6-4e714711cc50.crt`. Transfer this file back to the server:

```bash
$ scp pki/issued/e8b4e806-ac48-4cd5-b7e6-4e714711cc50.crt johndoe@[OPENVPN_SERVER_IP]:/tmp
```

Back on your **OpenVPN Server**, copy the client certificate to the `~/client-configs/keys/` directory:

```bash
$ cp /tmp/e8b4e806-ac48-4cd5-b7e6-4e714711cc50.crt ~/client-configs/keys/
```

Next, copy the `ca.crt` and `ta.key` files to the `~/client-configs/keys/` directory as well, and set the appropriate permissions for your sudo user:

```bash
$ cp ~/easy-rsa/ta.key ~/client-configs/keys/
$ sudo cp /etc/openvpn/server/ca.crt ~/client-configs/keys/
$ sudo chown johndoe.johndoe ~/client-configs/keys/*
```

With that, your server and client’s certificates and keys have all been generated and are stored in the appropriate directories on your **OpenVPN Server**.

## Step 8 — Configuring OpenVPN

OpenVPN, like many other popular open-source tools, offers a wide array of configuration options to tailor your server to your unique requirements.

First, copy the sample `server.conf` file as a starting point for your own configuration file:

```bash
$ sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/
```

Open the new file for editing with a text editor:

```bash
$ sudo vi /etc/openvpn/server/server.conf
```

You will need to change a few lines in this file. First, find the HMAC section of the configuration by searching for the `tls-auth` directive. This line will be enabled by default. Comment it out by adding a `;` to the beginning of the line. Then add a new line after it containing the value `tls-crypt ta.key` only:

```bash
;tls-auth ta.key 0
tls-crypt ta.key
```

Next, find the section on cryptographic ciphers by looking for the cipher lines. The default value is set to `AES-256-CBC`, however, the `AES-256-GCM` cipher offers a better level of encryption, performance, and is well supported in up-to-date OpenVPN clients. Comment out the default value by adding a `;` sign to the beginning of this line, and then add another line after it containing the updated value of `AES-256-GCM`:

```bash
;cipher AES-256-CBC
cipher AES-256-GCM
```

Right after this line, add an auth directive to select the HMAC message digest algorithm. For this, `SHA512` is a good choice:

```bash
auth SHA512
```

Next, find the line containing a `dh` directive, which defines Diffie-Hellman parameters. Since you configured all the certificates to use ECC, there is no need for a Diffie-Hellman seed file. Comment out the existing line that looks like `dh dh2048.pem` or `dh dh.pem`. The filename for the Diffie-Hellman key may be different than what is listed in the example server configuration file. Then add a line after it with the contents `dh none`:

```bash
;dh dh2048.pem
dh none
```

Next, OpenVPN should run with no privileges once it has started, so you will need to tell it to run with a user `nobody` and group `nogroup`. To enable this, find and uncomment the `user nobody` and `group nogroup` lines by removing the `;` sign from the beginning of each line:

```bash
user nobody
group nogroup
```

The settings above will create the VPN connection between your client and server, but will not force any connections to use the tunnel. If you wish to use the VPN to route all of your client traffic over the VPN, you will likely want to push some extra settings to the client computers.

To get started, find and uncomment the line containing push `redirect-gateway def1 bypass-dhcp`. Doing this will tell your client to redirect all of its traffic through your **OpenVPN Server**. Be aware that enabling this functionality can cause connectivity issues with other network services, like SSH:

```bash
push "redirect-gateway def1 bypass-dhcp"
```

Just below this line, find the `dhcp-option` section. Again, remove the `;` from the beginning of both of the lines to uncomment them:

```bash
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```

These lines will tell your client to use the free OpenDNS resolvers at the listed IP addresses. If you prefer other DNS resolvers you can substitute them in place of the used IPs.

This will assist clients in reconfiguring their DNS settings to use the VPN tunnel as the default gateway.

By default, the **OpenVPN Server** uses port 1194 and the UDP protocol to accept client connections. If you need to use a different port because of restrictive network environments that your clients might be in, you can change the port option. If you are not hosting web content on your **OpenVPN Server**, port 443 is a popular choice since it is usually allowed through firewall rules.

To change OpenVPN to listen on port 443, open the `server.conf`file and find the line that looks like this:

```bash
;port 1194
port 443
```

Oftentimes, the protocol is restricted to that port as well. If so, find the proto line below the port line and change the protocol from `udp` to `tcp`:

```bash
;proto udp
proto tcp
```

If you do switch the protocol to TCP, you will need to change the `explicit-exit-notify` directive’s value from `1` to `0`, as this directive is only used by UDP. Failing to do so while using TCP will cause errors when you start the OpenVPN service.

Find the `explicit-exit-notify` line at the end of the file and change the value to `0`:

```bash
explicit-exit-notify 0
```

If you have no need to use a different port and protocol, it is best to leave these settings unchanged.

If you selected a different name during the `./easyrsa gen-req `server command earlier, modify the `cert` and `key` lines in the `server.conf` configuration file so that they point to the appropriate `.crt` and `.key` files. If you used the default name, server, this is already set correctly:

```bash
ca ca.crt
cert openvpn.crt
key openvpn.key
```
When you are finished, save and close the file.

## Step 9 — Adjusting the **OpenVPN Server** networking configuration

There are some aspects of the server’s networking configuration that need to be tweaked so that OpenVPN can correctly route traffic through the VPN. The first of these is IP forwarding, a method for determining where IP traffic should be routed. This is essential to the VPN functionality that your server will provide.

To adjust your OpenVPN server’s default IP forwarding setting, open the `/etc/sysctl.conf` file using your preferred editor:

```bash
$ sudo vi /etc/sysctl.conf
```

Then add or change the following line:

```bash
net.ipv4.ip_forward = 1
```

To read the file and load the new values for the current session, type:

```bash
$ sudo sysctl -p
```

Now your **OpenVPN Server** will be able to forward incoming traffic from one ethernet device to another. This setting makes sure the server can direct traffic from clients that connect on the virtual VPN interface out over its other physical ethernet devices. This configuration will route all web traffic from your client via your server’s IP address, and your client’s public IP address will effectively be hidden.

## Step 10 — Firewall configuration

After successfully installing OpenVPN on your server, configuring it, and generating the necessary keys and certificates for client access, the next step involves directing incoming web traffic from clients. 

Despite completing these initial setup tasks, you have yet to specify how OpenVPN should manage client traffic. This can be accomplished through the implementation of firewall rules and routing configurations.

If you have followed the prerequisites outlined at the beginning of this guide, you should have `ufw` installed and operational on your server. Enabling masquerading is essential to allow OpenVPN traffic through the firewall. Masquerading, an iptables concept, facilitates dynamic **Network Address Translation (NAT)**, ensuring accurate routing of client connections.

Before opening the firewall configuration file to add the masquerading rules, you must first find the public network interface of your machine. Your public interface is the string found within this command’s output that follows the word `dev`. To do this, type:

```bash
$ ip route list default

default via 156.211.222.90 dev eth0 proto static
```
When you have the interface associated with your default route, open the `/etc/ufw/before.rules` file to add the relevant configuration:

```bash
$ sudo vi /etc/ufw/before.rules
```

UFW rules are typically added using the `ufw` command. Rules listed in the `before.rules` file, though, are read and put into place before the conventional UFW rules are loaded. Towards the top of the file, add the lines below. This will set the default policy for the `POSTROUTING` chain in the NAT table and masquerade any traffic coming from the VPN. Remember to replace `eth0` in the `-A POSTROUTING` line below with the interface you found in the above command:

```bash
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES
```

Save and close the file when you are finished.

Next, you need to tell UFW to allow forwarded packets by default as well. To do this, open the `/etc/default/ufw` file:

```bash
$ sudo vi /etc/default/ufw
```

Inside, find the `DEFAULT_FORWARD_POLICY` directive and change the value from `DROP` to `ACCEPT`:

```bash
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Save and close the file when you are finished.

Next, adjust the firewall itself to allow traffic to OpenVPN. If you did not change the port and protocol in the `/etc/openvpn/server.conf` file, you will need to open up `TCP` traffic to port `443`. If you modified the port and/or protocol, substitute the values you selected here.

In case you forgot to add the SSH port when following the prerequisite tutorial, add it here as well:

```bash
$ sudo ufw allow 443/tcp
$ sudo ufw allow 53/udp
$ sudo ufw allow 80/tcp
$ sudo ufw allow OpenSSH
```

If you are using a different firewall or have customized your UFW configuration, you may need to add additional firewall rules. For example, if you decide to tunnel all of your network traffic over the VPN connection, you will need to ensure that port 53 traffic is allowed for DNS requests, and ports like 80 and 443 for HTTP and HTTPS traffic respectively. If there are other protocols that you are using over the VPN then you will need to add rules for them as well.

After adding those rules, disable and re-enable UFW to restart it and load the changes from all of the files you’ve modified:

```bash
$ sudo ufw disable
$ sudo ufw enable
```

Your server is now configured to correctly handle OpenVPN traffic. With the firewall rules in place, you can start the OpenVPN service on the server.

## Step 11 — Starting OpenVPN

OpenVPN runs as a `systemd` service, so you can use `systemctl` to manage it. You will configure OpenVPN to start up at boot so you can connect to your VPN at any time as long as your server is running. To do this, enable the OpenVPN service by adding it to `systemctl`:

```bash
$ sudo systemctl -f enable openvpn-server@server.service
```

Then start the OpenVPN service:

```bash
$ sudo systemctl start openvpn-server@server.service
```

Double check that the OpenVPN service is active with the following command. You should see active (running) in the output:

```bash
$ sudo systemctl status openvpn-server@server.service
```

## Step 12 — Creating the client configuration infrastructure

Creating configuration files for OpenVPN clients can be somewhat involved. Each client requires its own configuration file, which must align with the settings outlined in the server’s configuration file. Instead of creating a single configuration file usable for only one client, this process establishes a client configuration infrastructure. This infrastructure enables the generation of config files on-the-fly. 

Initially, you will craft a “base” configuration file. Subsequently, you will develop a script to generate unique client config files, certificates, and keys as required.

### Creating a base configuration file

Get started by creating a new directory where you will store client configuration files within the client-configs directory you created earlier:

```bash
$ mkdir -p ~/client-configs/files
```

Next, copy an example client configuration file into the client-configs directory to use as your base configuration:

```bash
$ cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf
```

Open this new file using your preferred text editor:

```bash
$ vi ~/client-configs/base.conf
```

Inside, locate the remote directive. This points the client to your **OpenVPN Server** address — the public IP address of your **OpenVPN Server**. If you decided to change the port that the **OpenVPN Server** is listening on, you will also need to change `443` to the port you selected:

```bash
remote [OPENVPN_SERVER_IP] 443
```

Be sure that the protocol matches the value you are using in the server configuration:

```bash
proto tcp
```

Next, uncomment the user and group directives by removing the `;` sign at the beginning of each line:

```bash
user nobody
group nogroup
```

Find the directives that set the `ca`, `cert`, and `key`. Comment out these directives since you will add the certs and keys within the file itself shortly:

```bash
;ca ca.crt
;cert client.crt
;key client.key
```

Similarly, comment out the `tls-auth` directive, as you will add `ta.key` directly into the client configuration file (and the server is set up to use `tls-crypt`):

```bash
;tls-auth ta.key 1
```

Mirror the cipher and auth settings that you set in the `/etc/openvpn/server/server.conf` file:

```bash
cipher AES-256-GCM
auth SHA512
```

Next, add the `key-direction` directive somewhere in the file. You must set this to `1` for the VPN to function correctly on the client machine:

```bash
key-direction 1
```

Specify that the client should not cache username or password:

```bash
auth-nocache
```

Specify that the client is TLS-capable and assume the client role during the TLS handshake:

```bash
tls-client
```

Specify the utilized cipher suites for secure communication:

```bash
tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384
```

Disable logs on the client-side (only errors will be visible):

```bash
verb 0
```

Finally, add a few commented out lines to handle various methods that Linux based VPN clients will use for DNS resolution. You will add two similar, but separate sets of commented out lines.

The first set is for clients that do not use `systemd-resolved` to manage DNS. These clients rely on the `resolvconf` utility to update DNS information for Linux clients.

```bash
;script-security 2
;up /etc/openvpn/update-resolv-conf
;down /etc/openvpn/update-resolv-conf
```

Now add another set of lines for clients that use `systemd-resolved` for DNS resolution:

```bash
; script-security 2
; up /etc/openvpn/update-systemd-resolved
; down /etc/openvpn/update-systemd-resolved
; down-pre
; dhcp-option DOMAIN-ROUTE .
```

Save and close the file when you are finished.

### Creating a script for generating client configuration

Next, you will create a script that will compile your base configuration with the relevant certificate, key, and encryption files and then place the generated configuration in the `~/client-configs/files` directory. Open a new file called `make_config.sh` within the `~/client-configs` directory:

```bash
$ vi ~/client-configs/make_config.sh
```

Inside, add the following content:

```bash
#!/bin/bash
 
# First argument: Client identifier
 
KEY_DIR=~/client-configs/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf
 
cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n\n<tls-crypt>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-crypt>\n') \
    > ${OUTPUT_DIR}/${1}.ovpn
```

Save and close the file when you are finished.

Before moving on, be sure to mark this file as executable by typing:

```bash
$ chmod 700 ~/client-configs/make_config.sh
```

This script automates the process of creating a client configuration file by consolidating all necessary information into one place. It copies the `base.conf` file you have created, gathers the certificate and key files associated with your client, extracts their contents, merges them with the base configuration, and produces a new client configuration file. 

By utilizing this method, you streamline the management of client configurations, certificates, and keys. This ensures that crucial information is conveniently stored in a single location. 

It's important to note that each time you onboard a new client, you must generate new keys and certificates before executing this script to create their configuration file. This script will be an integral part of your workflow for adding new clients, providing you with a quick and efficient means to organize and generate configuration files.

## Step 13 — Generating client configurations

If you followed along with the guide, you created a client certificate and key named `e8b4e806-ac48-4cd5-b7e6-4e714711cc50.crt` and `e8b4e806-ac48-4cd5-b7e6-4e714711cc50.key`, respectively. You can generate a config file for these credentials by moving into your `~/client-configs` directory and running the script you made at the end of the previous step:

```bash
$ cd ~/client-configs
$ ./make_config.sh e8b4e806-ac48-4cd5-b7e6-4e714711cc50
```

This will create a file named `e8b4e806-ac48-4cd5-b7e6-4e714711cc50.ovpn` in your `~/client-configs/files` directory:

```bash
$ ls ~/client-configs/files
```

You need to transfer this file to the device you plan to use as the client. For instance, this could be your local computer or a mobile device.

## Step 14 — Installing the client configuration

This section covers how to install a client VPN profile on Linux and Android. None of these client instructions are dependent on one another, so feel free to skip to whichever is applicable to your device.

The OpenVPN connection will have the same name as whatever you called the `.ovpn` file. In regards to this tutorial, this means that the connection is named `e8b4e806-ac48-4cd5-b7e6-4e714711cc50.ovpn`, aligning with the first client file you generated.

### Linux

#### Installing

If you are using Linux, there are a variety of tools that you can use depending on your distribution. Your desktop environment or window manager might also include connection utilities.

The most universal way of connecting, however, is to just use the OpenVPN software.

On Ubuntu or Debian, you can install it just as you did on the server by typing:

```bash
$ sudo apt install openvpn
```

#### Configuring clients that use systemd-resolved

First determine if your system is using systemd-resolved to handle DNS resolution by checking the `/etc/resolv.conf` file:

```bash
$ cat /etc/resolv.conf
```

If your system is configured to use `systemd-resolved` for DNS resolution, the IP address after the `nameserver` option will be `127.0.0.53`. There should also be comments in the file like the output that is shown that explain how systemd-resolved is managing the file. If you have a different IP address than `127.0.0.53` then chances are your system is not using `systemd-resolved` and you can go to the next section on configuring Linux clients that have an `update-resolv-conf` script instead.

To support these clients, first install the `openvpn-systemd-resolved` package. It provides scripts that will force `systemd-resolved` to use the VPN server for DNS resolution:

```bash
$ sudo apt install openvpn-systemd-resolved
```

Once that package is installed, configure the client to use it, and to send all DNS queries over the VPN interface.

Open the client’s VPN file:

```bash
$ vi e8b4e806-ac48-4cd5-b7e6-4e714711cc50.ovpn
```

Now uncomment the following lines that you added earlier:

```bash
script-security 2
up /etc/openvpn/update-systemd-resolved
down /etc/openvpn/update-systemd-resolved
down-pre
dhcp-option DOMAIN-ROUTE .
```

If your client uses `systemd-resolved` to manage DNS, check the settings are applied correctly by running the `systemd-resolve --status` command like this:

```bash
$ systemd-resolve --status tun0

Link 22 (tun0)
. . .
         DNS Servers: 208.67.222.222
                      208.67.220.220
          DNS Domain: ~.
```

If you see the IP addresses of the DNS servers that you configured on the **OpenVPN Server**, along with the `~.` setting for DNS Domain in the output, then you have correctly configured your client to use the VPN server’s DNS resolver. You can also check that you are sending DNS queries over the VPN by using a site like DNS Leak.

#### Configuring clients that use update-resolv-conf

If your system is not using `systemd-resolved` to manage DNS, check to see if your distribution includes an `/etc/openvpn/update-resolv-conf` script instead:

```bash
$ ls /etc/openvpn
```

If your client includes the `update-resolv-conf` file, then edit the OpenVPN client configuration file that you transferred earlier:

```bash
$ vi e8b4e806-ac48-4cd5-b7e6-4e714711cc50.ovpn
```

Uncomment the three lines you added to adjust the DNS settings:

```bash
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

#### Connecting

```bash
$ sudo openvpn --config e8b4e806-ac48-4cd5-b7e6-4e714711cc50.ovpn
```

This should connect you to your VPN Server.

## Step 14 — Testing VPN connection

Once everything is installed, a simple check confirms everything is working properly. Without having a VPN connection enabled, open a browser and go to [DNS Leak Test](https://www.dnsleaktest.com/) website.

The site will return the IP address assigned by your internet service provider and as you appear to the rest of the world. To check your DNS settings through the same website, click on **Extended Test** and it will tell you which DNS servers you are using.

If you see a country that is different from where you are currently then you have configured everything correctly.

## Step 15 - Revoking client certificates

Occasionally, you may need to revoke a client certificate to prevent further access to the **OpenVPN Server**.

The revocation process is done on the **CA Server**, so you need to execute the following commands there.

To revoke a certificate, navigate to the `easy-rsa` directory on your **CA Server**:

```bash
$ cd ~/easy-rsa
```

Next, run the `easyrsa` script with the `revoke` option, followed by the client name you wish to revoke. Following the practice example above, the CN of the certificate is `e8b4e806-ac48-4cd5-b7e6-4e714711cc50`:

```bash
$ ./easyrsa revoke e8b4e806-ac48-4cd5-b7e6-4e714711cc50
```

This will ask you to confirm the revocation by entering `yes`.

After confirming the action, the CA will revoke the certificate. However, remote systems that rely on the CA have no way to check whether any certificates have been revoked. Users and servers will still be able to use the certificate until the CA’s **Certificate Revocation List (CRL)** is distributed to all systems that rely on the CA.

Now that you have revoked a certificate, it is important to update the list of revoked certificates on your CA server. Once you have an updated revocation list you will be able to tell which users and systems have valid certificates in your CA.

To generate a CRL, run the `easy-rsa` command with the `gen-crl` option while still inside the `~/easy-rsa` directory:

```bash
$ ./easyrsa gen-crl
```

If you have used a passphrase when creating your `ca.key` file, you will be prompted to enter it. The `gen-crl` command will generate a file called `crl.pem`, containing the updated list of revoked certificates for that CA.

Next you’ll need to transfer the updated `crl.pem` file to all servers and clients that rely on this CA each time you run the `gen-crl` command. Otherwise, clients and systems will still be able to access services and systems that use your CA, since those services need to know about the revoked status of the certificate.

You can do it using SCP command we used before to upload CRL to our **OpenVPN Server**:

```bash
$ scp ~/easy-rsa/pki/crl.pem johndoe@[OPENVPN_SERVER_IP]:/tmp
```

Once you have revoked a certificate for a client using those instructions, you will need to copy the generated `crl.pem` file to your OpenVPN server in the `/etc/openvpn/server` directory:

```bash
$ sudo cp /tmp/crl.pem /etc/openvpn/server/
```

Next, open the OpenVPN server configuration file:

```bash
$ sudo vi /etc/openvpn/server/server.conf
```

At the bottom of the file, add the `crl-verify` option, which will instruct the **OpenVPN Server** to check the certificate revocation list that you created each time a connection attempt is made:

```bash
crl-verify crl.pem
```

Save and close the file.

Finally, restart OpenVPN to implement the certificate revocation:

```bash
$ sudo systemctl restart openvpn-server@server.service
```

The client should no longer be able to successfully connect to the server using the old credential.

You can use this process to revoke any certificates that you have previously issued for your server.

# References

- [How To Set Up and Configure an OpenVPN Server on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-22-04)
- [OpenVPN Community Resources - 2x HOW TO](https://openvpn.net/community-resources/how-to/)