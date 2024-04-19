# Initial Server Setup With Ubuntu

## Prerequisites

The utilized OS version is Ubuntu 22.04 (LTS) x64 with 1 CPU, 1 GB RAM and 25 GB disk.

When you first create a new Ubuntu server, you should perform some important configuration steps as part of the initial setup. These steps will increase the security and usability of the server and will give you a solid foundation for subsequent actions.

## Step 1 - Getting a server

The first step is to get a server. You can either use you own physical server or rent it using one of the popular cloud providers like **Amazon**, **Cloudflare** or **DigitalOcean**.

## Step 2 - Finding server's public IP address

The next step is to connect to the server but to log in to it you need to know its **public IP address**.

Very often cloud providers allow you to view it via a control panel or if you are already singed in you can view it using `ifconfig` command (elevated privileges are required):

```bash
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 152.233.17.47  netmask 255.255.240.0  broadcast 134.209.255.255
        inet6 fe80::c09d:5dff:febf:cac7  prefixlen 64  scopeid 0x20<link>
        ether c2:9d:5d:bf:ca:c7  txqueuelen 1000  (Ethernet)
        RX packets 623650  bytes 643873025 (643.8 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 702408  bytes 422487311 (422.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.114.0.2  netmask 255.255.240.0  broadcast 10.114.15.255
        inet6 fe80::7460:36ff:fe7e:f5ac  prefixlen 64  scopeid 0x20<link>
        ether 76:60:36:7e:f5:ac  txqueuelen 1000  (Ethernet)
        RX packets 70  bytes 4900 (4.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 379  bytes 18394 (18.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 910  bytes 98020 (98.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 910  bytes 98020 (98.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.8.0.1  netmask 255.255.255.255  destination 10.8.0.2
        inet6 fe80::f1ee:17fa:f3ed:d208  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 219178  bytes 155367774 (155.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 231983  bytes 193349260 (193.3 MB)
        TX errors 0  dropped 1761 overruns 0  carrier 0  collisions 0
```

### Why there are multiple interfaces?

Generally, there are two types of interfaces: **public** and **private**.

Public interfaces are intended for communication with the outer world, while private interfaces are used for communication within a private network and are not accessible from the Internet.

This differentiation allows isolating certain servers from public access while still enabling communication with other servers inside the same private network. It facilitates the construction of something called a Virtual Private Cloud (VPC).

In the previous example, there is one public interface `eth0`, two private interfaces `eth1` and `tun0`, and one special interface `lo` called **loopback**, which is usually used for testing purposes on a local machine because it is always available.

Some interfaces can be virtual, meaning that they may not physically exist on the server, but the operating system still treats them as real.

Of course, nowadays, almost all servers rented from cloud providers are virtual, and all physical components are emulated; hence, technically, all interfaces on such machines are virtual, but it doesn't matter in this case.

To find out the public IP address of our server, we need to understand private IP addresses and their possible ranges.

Long story short, they usually have one of the following structures: `10.x.x.x`, `172.16.x.x`, and `192.168.x.x`.

Considering all of the above, we have only one IP address that doesn't look like a private address, and it is `152.233.17.47` on interface `eth0`.

## Step 3 - Logging via SSH

### Nice to meet you, I am SSH

If the server is remote, then you will need to connect to it using a terminal.

**SSH protocol** is the primary method for connecting to remote servers.

SSH supports two types of user authentication: **password-based** and **key-based**.

With password-based SSH authentication, you connect by entering a username and password. Key-based authentication, on the other hand, allows you to connect using a secret key, also known as a private key.

Password-based authentication is simpler to use but less secure because you must enter the password each time you log in, which can be intercepted.

Keys, however, are typically much longer than regular passwords and are based on strong cryptographic algorithms, making them significantly harder to crack and more secure in most scenarios.

Therefore, if you are currently using password-based authentication, I strongly recommend setting up key-based authentication and disabling password-based authentication entirely for enhanced security.

### How to connect to a server

In order to connect to the server, you need to know its public IP address and a user existing in the system.

Then you can use the command below to connect to the remote server:

```bash
$ ssh [USERNAME]@[SERVER_IP]
```

Usually, after that, you will be prompted to accept a key which will allow you to verify the remote server. After accepting, you will be prompted to enter a password for the user or for the private key if you are using key-based authentication.

Most of the time, the operating system will not have any user except one - **root**.

### Nice to meet you, I am root

The **root** user is the administrative user in a Linux environment with elevated privileges. Due to the extensive privileges associated with the **root** account, it is not recommended for regular use. The root account has the capability to make highly destructive changes, even unintentionally.

For this reason, the **root** account is primarily used during the initial server setup. Subsequently, a new account should be created and granted administrative privileges to replace the use of the **root** account in regular operations.

## Step 4 - Creating a new user

Once you log in as **root**, you’ll be able to add the new user account. In the future, you must log in with this new account instead of **root**.

The command below allows you to create a new user:

```bash
$ adduser [USERNAME]
```

You will be prompted with a series of questions, beginning with setting the account password. Enter a strong password and, if desired, provide any optional additional information. This extra information is not mandatory, and you can simply press `ENTER` to skip any fields you prefer.

## Step 5 - Granting administrative privileges

Now you have a new user account with regular privileges. However, there will be times when you need to perform administrative tasks that require **root** access.

To avoid constantly switching between your regular user and the **root** account, you can grant your user account superuser privileges. This allows your normal user to execute commands with administrative permissions by using the `sudo` command.

To grant these privileges to your new user, you will need to add the user to the `sudo` system group. By default, on Ubuntu, members of the `sudo` group are permitted to use the `sudo` command.

To add your new user to the `sudo` group, execute the following command as **root** (replace `[USERNAME]` with your new user's username):

```bash
$ usermod -aG sudo [USERNAME]
```

Now, you can prepend `sudo` before commands to execute them with superuser privileges while logged in as your regular user.

## Step 6 - Setting up a firewall

A firewall acts like a protective barrier for your server, preventing unwanted connections and specifying which connections are permissible.

Configuring a firewall can be complex, especially for beginners, but there are some essential configurations that should be enabled during the initial setup.

Ubuntu servers can utilize **UFW (aka Uncomplicated Firewall)** to control which connections to specific services are permitted. You can establish a basic firewall using this utility.

When applications are installed, they can register their profiles with UFW, enabling **UFW** to manage them by name. For instance, OpenSSH, which facilitates server connections, comes with a pre-registered profile in **UFW**.

To view a list of installed **UFW** profiles, use the following command:

```bash
$ ufw app list

Output
Available applications:
  OpenSSH
```

Ensure that your firewall allows SSH connections so that you can log into your server in the future. Allow these connections by entering:

```bash
$ ufw allow OpenSSH
```

Now enable the firewall by typing:

```bash
$ ufw enable
```

Type `y` and press `ENTER` to proceed. You can see that SSH connections are still allowed by typing:

```bash
$ ufw status

Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

At present, the firewall blocks all connections except for SSH. If you install and configure additional services in the future, remember to adjust the firewall settings to accommodate new traffic into your server.

## Step 7 - Enabling external access for the regular user

Now that you have a regular user set up for daily use, the next step is to enable SSH access directly to this account.

To configure SSH access for your new user, the process depends on whether your server's **root** account uses password-based authentication or SSH keys for authentication.

### If the root account uses password based authentication

If you logged in to your root account using a password then password authentication is enabled for SSH. You can SSH to your new user account by opening up a new terminal session and using SSH with your new username:

```bash
$ sudo [COMMAND]
```

You will receive a prompt for your regular user’s password when using sudo for the first time each session (and periodically afterward).

### If the root account uses key based authentication

If you logged in to your **root** account using SSH keys, then the password authentication is disabled for SSH. To log in as your regular user with an SSH key, you need to add your local public key to your new user's `~/.ssh/authorized_keys` file.

Since your public key is already in the **root** account's `~/.ssh/authorized_keys` file on the server, you can copy that file and directory structure to your new user account using the current session.

The easiest way to copy the files with the correct ownership and permissions is with the `rsync` command.

The `rsync` command treats sources and destinations that end with a trailing slash differently than those without a trailing slash. When using `rsync` below, ensure that the source directory `~/.ssh` does not include a trailing slash (check to make sure you are not using `~/.ssh/`).

If you accidentally add a trailing slash to the command, rsync will copy the contents of the root account’s `~/.ssh` directory to the `sudo` user’s home directory instead of copying the entire `~/.ssh` directory structure. The files will be in the wrong location and SSH will not be able to find and use them.

This command will copy the **root** user's `.ssh` directory, preserve permissions, and modify file owners in a single step. Be sure to replace `[USERNAME]` in the command below with your regular user's name:

```bash
$ rsync --archive --chown=[USERNAME]:[USERNAME] ~/.ssh /home/[USERNAME]
```

Now, open up a new terminal session on your local machine, and use SSH with your new username:

```bash
$ ssh [USERNAME]@[SERVER_IP]
```

You should be connected to your server with the new user account without using a password. Remember, if you need to run a command with administrative privileges, type sudo before the command like this:

```bash
$ sudo [COMMAND]
```

You will be prompted for your regular user’s password when using `sudo` for the first time each session (and periodically afterward).