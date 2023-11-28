# Self-hosting the Bitwarden Password Manager in a LAN using the Vaultwarden Backend on a Raspberry Pi 3 Model A+

## Introduction

Bitwarden is an open source freemium password manager that provides unlimited storage of logins and passwords in a vault that can be access on multiple devices. It generates strong passwords and uses AES-256 bit end-to-end encryption for secure communication between devices. Users have the option to either use the service online over the cloud or using a self-hosted network. Bitwarden's self-hosted version is good, but it blocks premium features behind a paywall, and requires the host user to create a Bitwarden account. A community-made solution named Vaultwarden provides a self-hosted version of Bitwarden with all its premium features, and it doesn't require the host user to create a Bitwarden account to use.

This project self-hosts a Bitwarden server only accessible via LAN, using the Vaultwarden backend hosted on a Raspberry Pi 3 Model A+.

## Requirements

No specific hardware is required for this system. It is recommended to have a personal/work computer for configuration and testing, and a server computer to run the Vaultwarden web server. For this project, I used a Windows 11 PC and a Raspberry Pi 3 Model A+ for the server computer.

## Sections

Each main section of the project is outlined in a separate markdown file. Click one of the following headers to redirect to that markdown file.

> NOTE - Files created in this project can be found in the directory of the section they have been created in

### [Setting up the Raspberry Pi](./contents/raspberry_pi_setup/)

Setting up the Raspberry Pi to work as a server

### [Setting up SSH](./contents/ssh_setup/)

Setting up SSH for remote connection via PC

### [Setting up Docker on the Raspberry Pi](./contents/docker_setup/)

Setting up Docker on the Raspberry Pi for running containerised applications

### [Setting up HTTPS for Vaultwarden Server](./contents/https_setup/)

Setting up a domain name, a private DNS server and SSL certificates for HTTPS communication for the Vaultwarden server
