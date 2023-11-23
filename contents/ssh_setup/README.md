# Setting up SSH
A very useful feature that comes with all operating systems is Secure Shell (SSH), which is a network comunnication protocol that allows two computers to communicate with each other. With this, you can access the Raspberry Pi from your PC through the command line. Secure File Transfer Protocol (SFTP) is an extension of SSH, and is a network protocol for securely accessing and transferring files between two computers. This allows you to create and edit files on your PC, and then transfer them to your Raspberry Pi after completion.

Since I was going to do the majority of development via my PC, I ensured that SSH and SFTP were configured correctly and securely.

## Initial SSH Setup
By default, SSH comes with Raspbian, but to make sure, enter:

```shell
which ssh
```
