# Setting up SSH

A very useful feature that comes with all operating systems is Secure Shell (SSH), which is a network comunnication protocol that allows two computers to communicate with each other. With this, you can access the Raspberry Pi from your PC through the command line. Secure File Transfer Protocol (SFTP) is an extension of SSH, and is a network protocol for securely accessing and transferring files between two computers. This allows you to create and edit files on your PC, and then transfer them to your Raspberry Pi after completion.

Since I was going to do the majority of development via my PC, I ensured that SSH and SFTP were configured correctly and securely.

## Initial SSH Setup

By default, SSH comes with Raspbian, but to make sure, enter:

```shell
# Shows directory of ssh binary file
which ssh
```

If a directory doesn't appear, then SSH isn’t installed on the Raspberry Pi. To install SSH, enter the following commands one by one:

```shell
# Update and upgrade system
sudo apt update && sudo apt upgrade -y
# Install SSH server
sudo apt install openssh-server
# Validate install
which ssh
```

After confirming that SSH is installed, your Raspberry Pi needs some basic security.

Firstly, it is good practice to create a sudo user, so that you don't accidentally make fatal changes to your system while logged in as root. Check to see if there are any extra users:

```shell
# Returns name of custom users
getent passwd {1000..60000} | cut -d: -f1
```

If there is a result, then you can move on to the next step. If not, create an account by entering:

```bash
# Add user (replace '<username>')
useradd -m <username>
# Create a password for new user
passwd <username>
```

`passwd` prompts you to set the password for the new user. Then give the new user root priviledges:

```shell
# Add user to sudo group
usermod -aG sudo <username>
```

Once you have a sudo user available, make sure you're logged into the user account and go to the SSH server configuration directory:

```shell
cd /etc/ssh/sshd_config.d
```

Create a config file to add SSH configurations for added security. Make sure the file ends in `.conf`.

```shell
sudo nano custom_config.conf
```

The terminal will show Nano, the default Raspbian text editor, inside a newly created file. Add the following lines to the file:

```bash
# Change port from default port 22 to different number
Port 2222
# Onlly allow SSH connections from users logging in as your user in your LAN
AllowUsers username@192.168.0.0/16
# Don't allow users to directly log into root when establishing SSH connection
PermitRootLogin no
```

Save the file with <kbd>CTRL + O</kbd>, <kbd>ENTER</kbd> and exit the file with <kbd>CTRL + X</kbd>.

To make sure that the SSH server is configured correctly without any errors, check to see if the SSH service is active:

```shell
sudo systemctl status ssh
```

If so, the 'Active' line will say 'active (running)' in green.

<p align="center">
<img src="./images/ssh_service_status.jpg" alt="Raspberry Pi Default Home Screen" height=50px>
</p>

If it is active, restart the service:

```shell
sudo systemctl restart ssh
```

Otherwise start the SSH service:

```shell
sudo systemctl start ssh
```

Check to see if the service is running without any errors:

```shell
sudo systemctl status ssh
```

The 'Active' line should say 'active (running)' in green.