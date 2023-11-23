# Self-hosting Bitwarden in LAN using Vaultwarden Backend on Raspberry Pi
Bitwarden is an open source freemium password manager that provides unlimited storage of logins and passwords in a vault that can be access on multiple devices. It generates strong passwords and uses AES-256 bit end-to-end encryption for secure communication between devices. Users have the option to either use the service online over the cloud or using a self-hosted network. Bitwarden's self-hosted version is good, but it blocks premium features behind a paywall, and requires the host user to create a Bitwarden account. A community-made solution named Vaultwarden provides a self-hosted version of Bitwarden with all its premium features, and it doesn't require the host user to create a Bitwarden account to use.

This project self-hosts a Bitwarden server only accessible via LAN, using the Vaultwarden backend hosted on a Raspberry Pi 3 Model A+.

## Requirements
No specific hardware is required for this system. It is recommended to have a personal/work computer for configuration and testing, and a server computer to run the Vaultwarden web server. For this project, I used a Windows 11 PC and a Raspberry Pi 3 Model A+ for the server computer.

## Contents