# frcjenkins.wpi.edu

In order to test that the software WPILib writes is compatible with the FIRST Robotics Control System hardware we manage and maintain frcjenkins.wpi.edu (frcjenkins). frcjenkins is a physical computer that that lives in WPI AK 120D. Attached to the computer via ethernet is the teststand that has the major control system components, just like an actual robot. The name of the system comes from the old service that would run the continuous integration system for WPILib, named Jenkins. Even though we do not use Jenkins anymore, we kept the name.

When a commit is made to selected WPILib code repositories, a process is triggered that will complete a specified set of actions. A similar, but more secure process also occurs for pull requests made to selected repositories. Because anyone with a GitHub account can open a pull request, we be sure that settings are set so all code must be reviewed by a trusted WPILib Developer before running on the teststand.

# Hardware

<!-- Insert photo of hardware here. Include iMac and teststand -->

# OS Version and Install

Ubuntu 18.04

<!-- Link to Ubuntu install instructions -->

```bash
sudo apt update
sudo apt upgrade
```

# Initial Setup

## Creating user accounts

When first configuring frcjenkins, it is important to create individual user accounts for maintainers of the system and key WPILib leadership only. Do not allow root login or have a root password enabled.


```bash
# Add a user with name of username.
sudo adduser <username>
# Add that user to the sudo group; giving them permission to use sudo
sudo usermod -aG sudo <username>
```

## Configure Firewall (ufw)

Using a firewall is important to control the connections outside parties can make to a computer. frcjenkines uses the UFW (Uncomplicated Firewall) command line utility for controlling the firewall. In general, all inbound traffic should be dropped except for specific ports. This is the "allow-list" method – we only allow what we want and block everything else.

1. Make sure you have ufw installed.

```bash
sudo apt install ufw
```

2. Now that ufw is installed, we can check on its status by running:

```bash
sudo ufw status
```

3. If ufw responds with `Status: inactive`, you can enable it with:

```bash
sudo ufw enable
```

## Remote Access (sshd)

To allow permitted users to access the system, you can use SSH – Secure SHell. SSH allows users to login to a remote machine and execute commands on a remote machine. It is intended to provide secure encrypted communications between two hosts over an insecure network, like the internet.

To install the sshd server, run:

```bash
sudo apt install openssh-server
```

We must complete a few steps to make sure that an adversary is unable to use SSH to access the machine. All of these changes are going to be to the ssh server configuration file located at `/etc/ssh/sshd_conf`.

1. Disable root login via SSH.

```
PermitRootLogin no
```

2. Only allow users to authenticate with SSH keys, instead of passwords. Part of ensuring security with public private keys is to ask our users to create ssh keys that are secured with a passphrase. This means that if their personal device is compromised, an attacker cannot take their private key and use it to attack us.

```
PasswordAuthentication no
```

3. Now that we have updated our configuration settings we can apply the changes by restarting the ssh server.

```bash
sudo systemctl restart sshd
```

4. With the configuration applied, we can also modify the firewall to allow ssh connections.

```bash
sudo ufw allow ssh
```

5. Finally, we can add user's public keys to their ssh directories. Place the public key into the `/home/<user>/.ssh/authorized_keys` file and make sure the authorized key files has the appropriate permissions.

```bash
sudo chown <user>:<user> /home/<user>/.ssh/authorized_keys
sudo chmod 600 /home/<user>/.ssh/authorized_keys
```
Now you can test the connection by SSHing from your personal device into the server.

# Teststand Setup

## Image roboRIO

<!-- Link to frc-docs for imaging instructions -->

<!-- How to access collabnet and where to put the roboRIO image so the imaging tool can find it -->

## Network Connection

```bash
sudo vim 01-network-manager-all.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp2s0:
      dhcp4: yes
    enx041e64fc1848:
      addresses:
        - 10.1.90.5/24
```

```bash
sudo netplan apply
```

# Azure Setup

frcjenkins uses Azure Pipelines to queue jobs and manage requested runs from GitHub. Azure Pipelines also acts as a gate prevent unauthorized users from executing code on the server.

## Installing Dependencies

The Azure service runner requires Docker for some build steps. You must install Docker by following their guide available at: https://docs.docker.com/engine/install/ubuntu/.

## Creating A System User

We want to create a special user account to run the Azure agent service. This will allow us to give that user special permissions to do what it needs. What is important here is that this will be a system user – a user with no default home directory, no password, and is unable to login.

```bash
# Create a system user named azure
sudo useradd -r azure
# Create a home directory for the azure user and assign the correct permissions
sudo mkdir /var/lib/azure
sudo usermod -d /var/lib/azure azure
sudo chown -R azure:azure azure
```

## Configuring Azure Agent

1. Give azure system user permission to use Docker.

```bash
sudo usermod -aG docker azure
```

2. Follow Azure instructions to install the Azure agent as a service. It is important to do these steps while logged into the azure system user. While already connected to the server, log in as the azure user with:

```bash
sudo su azure
cd ~
```

<!-- Actually walk though the steps here -->

<!-- Screenshots of Azure -->

## Special Config Actions

```bash
modprobe loop
modprobe binfmt_misc
```
