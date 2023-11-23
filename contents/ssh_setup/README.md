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
<img src="./images/ssh_service_status.jpg" alt="SSH Status in systemctl" height=70px>
</p>

If it is active, restart the service:

```shell
sudo systemctl restart ssh
```

Otherwise start the SSH service:

```shell
sudo systemctl start ssh
```

Also make sure that the service is enabled, so that it runs on boot.

```shell
sudo systemctl enable ssh
```

Check to see if the service is running without any errors:

```shell
sudo systemctl status ssh
```

The 'Active' line should say 'active (running)' in green.

After confirming the SSH service is working, go to your PC and open PuTTY (see [this link](https://www.putty.org/) for download and installation). When you open PuTTY, you should be in the ‘Session’ category by default. In this tab, enter the IP address of the Raspberry Pi and the port number of the SSH connection. Also ensure that SSH is selected as the ‘Connection type’.

<p align="center">
<img src="./images/putty_config.jpg" alt="Adding IP and port to PuTTY" height=150px>
</p>

> NOTE – To find the private IPv4 address of your Raspberry Pi, enter ifconfig in the terminal, and you will find it under 'wlan0' (or 'eth0' if connected via Ethernet) as the 'inet' number

<p align="center">
<img src="./images/ifconfig.jpg" alt="ifconfig command" height=140px>
</p>

Name the configuration and then click ‘Save’ to save the configuration for your Raspberry Pi SSH connection.

<p align="center">
<img src="./images/putty_save.jpg" alt="Saving PuTTY configuration" width=300px>
<img src="./images/putty_load.jpg" alt="Loading PuTTY configuration" width=300px>
</p>

Test the SSH connection by clicking ‘Open’, then a terminal should appear asking to ‘login as:’. Enter the username of your user on the Raspberry Pi, and then the terminal will prompt you to enter a password. If entered correctly, you will now have access to your Raspberry Pi via your PC.

<p align="center">
<img src="./images/ssh_test_basic.jpg" alt="Testing SSH connection" height=170px>
</p>

## Setting up SSH Public Key Authentication

It is recommended that you use public key authentication when using SSH to prevent unauthorised users from brute forcing into the Raspberry Pi. This step makes it so that only users that provide the private key that decrypts data encrypted by the public key within the Raspberry Pi are given access to the machine.

For this step, I used Windows, if you want to know how to do this as a Mac or Linux user, see [this video](https://www.youtube.com/watch?v=GxRu35fy-oY) for the full guide I used.

To setup keys, go to the terminal of your PC and type:

```powershell
# Change the -C argument to a suitable name for the key
ssh-keygen -t ed25519 -C "<keyname>"
```

<p align="center">
<img src="./images/ssh_keygen_command.jpg" alt="Generating public authentication key" height=48px>
</p>

The terminal will prompt you to enter the directory and file name for the generated keys, enter the desired directory and file name you want to save the key as. I recommend saving it in the ‘.ssh’ directory in your home directory.

<p align="center">
<img src="./images/ssh_keygen_directory.jpg" alt="Changing name of SSH key" height=35px>
</p>

Enter a passphrase for the keys you will generate. This will add an extra layer of security. After adding the passphrase, you will be shown the key fingerprint and the key's randomart image:

<p align="center">
<img src="./images/ssh_keygen_complete.jpg" alt="Changing the name of SSH key" height=250px>
</p>

Then send the generated public key to the Raspberry Pi:

```powershell
# Change <key_name> to the name of your key, add -p flag if you changed your SSH port, and enter your Raspberry Pi's username and IP address
type $env:USERPROFILE\.ssh\<key_name>.pub | ssh -p <port_no> <pi_username>@<pi_ip_address> "cat >> .ssh/authorized_keys"
```

<p align="center">
<img src="./images/ssh_copy.jpg" alt="Copying public SSH key to Raspberry Pi" height=50px>
</p>

> NOTE - For Linux, 'ssh-copy-id' is a much simpler solution to use to copy your public key over to your Raspberry Pi:
>
> ```shell
> # Change <key_name> to the name of your key, add -p flag if you changed your SSH port, and enter your Raspberry Pi's username and IP address
> ssh-copy-id -i ~/.ssh/<key_name> -p <port_no> <pi_username>@<pi_ip_address>
> ```

After entering the password, the public key will be sent to the Raspberry Pi. You can find it in the `/home/$USER/.ssh/authorized_keys` file on your Raspberry Pi.

After confirming that the public key has been copied to your Raspberry Pi, update your SSH configuration file:

```shell
cd /etc/ssh/sshd_config.d
sudo nano custom_config.conf
```

Add the following lines to require users to login via public key authentication:

```bash
PasswordAuthentication no
PubkeyAuthentication yes
```

Save the changes and restart the SSH server.

```shell
sudo systemctl restart ssh
```
