# Configuring Vaultwarden

### Table of Contents

[Introduction](#introduction)

[Creating Admin Password](#creating-admin-password)

[Sections](#sections)

## Introduction

To make the Vaultwarden Docker container to allow emails to be sent to users and to allow more security for the application, you need to add multiple enviroment variables to the Docker Compose file.

This section outlines the steps I took to configure my Vaultwarden server on my Raspberry Pi.

## Creating Admin Password

Go to your Vaultwarden Docker Compose file directory and start the container:

```shell
# Go to directory
cd vaultwarden
# Start container
docker compose up -d
```

Firstly, you need to generate an admin token for Vaultwarden. To generate the token, you need to access the Vaultwarden container’s terminal. To do this, enter the terminal of the Docker container:

```shell
docker exec -it vaultwarden bash
```

<p align="center">
<img src="./images/docker_exec.jpg" alt="Accessing Vaultwarden's terminal" height=30px>
</p>

You will now have access to the container’s internal files and directories.

<p align="center">
<img src="./images/vaultwarden_terminal.jpg" alt="'ls' command in Vaultwarden's terminal" height=55px>
</p>

Enter the following to create a hash for the admin token:

```shell
./vaultwarden hash
```

<p align="center">
<img src="./images/vaultwarden_hash.jpg" alt="Generating a hash for admin password" height=70px>
</p>

Enter a strong password to use as the admin login. The script will ask you to re-enter the password after entering the first password.

The script will then create a hash for your password:

<p align="center">
<img src="./images/vaultwarden_generated_hash.jpg" alt="Generated hash for admin password" height=150px>
</p>

Copy the admin token and make sure to have it when editing the Vaultwarden Docker Compose file.

Exit the container’s terminal:

```shell
exit
```

Before adding the admin token as an environment variable, it is good practice to add environment variables in a separate file, so that sensitive information isn’t hard coded into the Docker Compose file and modifications to environment variables is much easier.

Create a .env file for Vaultwarden:

```shell
nano vaultwarden.env
```

Add the admin token environment variable:

```bash
# Hash of admin password dto use to login as admin
ADMIN_TOKEN=yourAdminToken
```

Remove the single quotes surrounding your admin token and add an extra dollar sign after each dollar sign. Your admin token environment variable should look like this:

<p align="center">
<img src="./images/vaultwarden_dotenv.jpg" alt="Generated hash for admin password" height=17px>
</p>

Save and exit, then edit the Docker Compose file:

```shell
nano docker-compose.yml
```

## Sections

#### Home Page: [Vaultwarden on Raspberry Pi](../../)

#### Previous Section: [Setting up HTTPS for Vaultwarden Server](../https_setup/)

#### Next Section: [...]()
