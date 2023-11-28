# Setting up HTTPS for Vaultwarden Server

### Table of Contents

[Introduction](#introduction)

[Acquiring a Public Domain](#acquiring-a-public-domain)

[Setting up Cloudflare DNS for the Public Domain](#setting-up-cloudflare-dns-for-the-public-domain)

[Setting up a Private DNS Server with dnsmasq](#setting-up-a-private-dns-server-with-dnsmasq)

[Creating HTTPS Domain Names with Nginx Proxy Manager and Cloudflare DNS](#creating-https-domain-names-with-nginx-proxy-manager-and-cloudflare-dns)

## Introduction

In order for Vaultwarden to work, it must be accessed via HTTPS, which requires these 4 main steps:

1. Acquiring a public domain
2. Setting up Cloudflare DNS for the public domain
3. Setting up a private DNS server with dnsmasq
4. Creating HTTPS domain names with Nginx Proxy Manager and Cloudflare DNS

## Acquiring a Public Domain

> NOTE - If you already have a public domain, then skip this step.

Buying a domain is recommended over using a free domain such as DuckDNS, as the domain ownership is tied to you only. I used Namecheap to buy a '.com' domain, so I will give a guide for buying a Namecheap domain.

On your PC, visit [Namecheap](https://www.namecheap.com/) and enter your desired domain name. I recommend using ‘.com’ as it is cheaper.

<p align="center">
<img src="./images/namecheap_search_domain.jpg" alt="Searching for an available domain on Namecheap" height=120px>
</p>

If the domain name is available, click ‘Add to cart’ and click ‘Checkout’.

Then you will be visited with the checkout page.

<p align="center">
<img src="./images/namecheap_checkout.jpg" alt="Checking out the domain in Namecheap" height=400px>
</p>

Make sure to enable ‘Domain Privacy’ to hide your information in the public Whois database. This database stores the information of a domain registrar.

You can optionally choose to auto-renew your domain registration and your domain privacy, of which the latter is free forever.

Create an account and enter your personal details, then your payment details. Then after confirming your order, you will have a public domain name for your own use.

## Setting up Cloudflare DNS for the Public Domain

For the DNS server, I used Cloudflare DNS as it is free, easy to setup, and is reputable. To start, create a free Cloudflare account on [Cloudflare](https://www.cloudflare.com).

Once you have signed up, see [this guide](https://www.namecheap.com/support/knowledgebase/article.aspx/9607/2210/how-to-set-up-dns-records-for-your-domain-in-cloudflare-account/), which will show you how to set the authoritative DNS servers for your Namecheap domain to Cloudflare DNS servers. If you didn’t buy a domain with Namecheap, simply search the domain registrar service and Cloudflare DNS, and you should find a guide telling you how to setup the authoritative DNS servers.

Once the Cloudflare DNS is set up, go to the records of your DNS server and add an A record. Give this a subdomain name of preferrably ‘home’, then add the private IP address of your Raspberry Pi in your LAN.

<p align="center">
<img src="./images/cloudflare_a_record_raspi.jpg" alt="Adding an A record for Raspberry Pi" height=250px>
</p>

Then create a CNAME record to point the domains of all of your private services (such as Nginx Proxy Manager and Vaultwarden) to your Raspberry Pi’s IP address.

<p align="center">
<img src="./images/cloudflare_cname_record_raspi.jpg" alt="Adding a CNAME record for Raspberry Pi" height=250px>
</p>

> NOTE - To add further security to your domain, add DNSSEC to protect against DNS spoofing. This means that when a client computer makes a request is made to a DNS server to resolve a domain name, the response is authenticated using public key cryptography to ensure that the response given to the client originates from the correct source.
>
> To do this, see [this guide](https://developers.cloudflare.com/dns/dnssec/) for a guide on enabling DNSSEC for your domain name.

## Setting up a Private DNS Server with dnsmasq

For the private Vaultwarden domain to resolve to the Vaultwarden web server that you will host on your Raspberry Pi, you need to create a private DNS server that can resolve private domain names. For this, I used dnsmasq as it is lightweight, and simple to setup for making a private DNS server.

On your Raspberry Pi, check if dnsmasq is installed:

```shell
which dnsmasq
```

<p align="center">
<img src="./images/which_dnsmasq.jpg" alt="'which dnsmasq' in terminal of Raspberry Pi" height=30px>
</p>

If it isn’t installed, firstly update and upgrade the Raspberry Pi, and then install dnsmasq:

```shell
sudo apt update && sudo apt upgrade -y
sudo apt install dnsmasq
```

Then go to /etc/dnsmasq.d/ and create a configuration file:

```shell
cd /etc/dnsmasq.d/
sudo nano dns_config.conf
```

In the text editor, enter the following into the file:

```conf
domain-needed
bogus-priv
no-resolv
server=primary.isp.dns.server
server=secondary.isp.dns.server
listen-address=::1,127.0.0.1,raspberry.pi.ip.address
```

Here is the meaning behind each line:

- domain-needed: Never forward A queries for plain domain names without dots or domain parts to upstream nameservers (e.g. the query for ‘mywebsite’ will not be forwarded past dnsmasq)
- bogus-priv: All reverse lookups for private IP address ranges e.g. 192.168.x.x will not be forwarded to upstream nameservers (e.g. the query ‘192.168.1.170’ will not be forwarded past dnsmasq)
- no-resolv: Don’t get upstream servers from /etc/resolv.conf file in Raspberry Pi
- server: Defines the upstream servers to use. Replace placeholders with the IPs of your ISP’s DNS servers
- listen-address: Defines which IP addresses to listen to when processing DNS queries. Replace the placeholder with your Raspberry Pi’s private IP address

Save the file, then start and enable the service, or restart the service if already started:

```shell
# Start and enable dnsmasq
sudo systemctl enable --now dnsmasq
# OR
# Restart dnsmasq
sudo systemctl restart dnsmasq
```

Check to see the status of dnsmasq:

```shell
sudo systemctl status dnsmasq
```

<p align="center">
<img src="./images/dnsmasq_status.jpg" alt="Checking the status of dnsmasq" height=65px>
</p>

To test to see if domains are resolving correctly use the 'nslookup' command:

```shell
nslookup www.google.com your.raspberry.pi.ip
```

<p align="center">
<img src="./images//nslookup_google.jpg" alt="Looking up Google to test DNS server" height=160px>
</p>

It should give a public IP address, which points to Google.

## Creating HTTPS Domain Names with Nginx Proxy Manager and Cloudflare DNS

For the Raspberry Pi to run multiple services and have each one be accessed with a domain name, it needs a reverse proxy. A reverse proxy is a part of a network that handles incoming connections coming from client devices outside the network to the servers within the network. Nginx Proxy Manager allows you to create unique domains for each service you run on your Raspberry Pi, for access via HTTP or HTTPS.

To run Nginx Proxy Manager, as well as other services on the Raspberry Pi, I used Docker Compose files, which allow you to define the dependencies of one or more Docker containers such as the image used, the environment variables, the port number for network connections, the volumes for data storage and more, all in one file. This makes it very easy to start, stop, and modify Docker containers.

On your Raspberry Pi, create a directory to keep the Docker Compose files for Nginx Proxy Manager and Vaultwarden:

```shell
# Making home server directory
mkdir home_server
cd home_server
# Making Nginx Proxy Manager directory
mkdir npm
cd npm
```

In the npm directory, create a Docker Compose file:

```shell
nano docker-compose.yml
```

Enter the following code:

```yaml
version: "3"
services:
  nginxproxymanager:
    container_name: nginx_proxy_manager
    image: "jc21/nginx-proxy-manager:latest"
    restart: unless-stopped
    ports:
      - "80:80"
      - "81:81"
      - "443:443"
    volumes:
      - data:/data
      - letsencrypt:/etc/letsencrypt
volumes:
  data:
  letsencrypt:
```

Save and exit the file. Then make sure you are in the same directory as the Docker Compose file and create a new Docker container using the Docker Compose file:

```shell
docker compose up -d
```

The output should look something like this:

<p align="center">
<img src="./images/docker_compose_up.jpg" alt="Running Docker containers with Docker Compose" height=110px>
</p>

Now, enter the IP address of your Raspberry Pi, followed by port 81 on your web browser like this:

<p align="center">
<img src="./images/npm_ip.jpg" alt="Navigating to Nginx Proxy Manager (NPM) via private IP address" height=60px>
</p>

You will be directed to the Nginx Proxy Manager Web Interface. Enter ‘admin@example.com’ as the email address and ‘changeme’ as the password. Then click ‘Sign In’

<p align="center">
<img src="./images/npm_login_first.jpg" alt="Logging into NPM for the first time" height=300px>
</p>

After signing in, you will be given a prompt to change your account details. You must change the email address and password to begin using NPM.

<p align="center">
<img src="./images/npm_change_email.jpg" alt="Changing the default email in NPM" height=300px>
</p>

<p align="center">
<img src="./images/npm_change_password.jpg" alt="Logging into NPM for the first time" height=370px>
</p>

After changing your account details, you will now be able to access the NPM web application. Before assigning IP addresses and ports to domain names, you need to get SSL certificates for your HTTPS websites.

To do this, firstly go to the ‘SSL Certificates’ tab in NPM.

<p align="center">
<img src="./images/npm_ssl_certificates_tab.jpg" alt="SSL Certificates Tab in NPM" height=90px>
</p>

<p align="center">
<img src="./images/npm_add_certificate.jpg" alt="Adding an SSL certificate in NPM" height=180px>
</p>

Click ‘Add SSL Certificate’, then enter the subdomain you entered in Cloudflare for your private network’s Nginx server, followed by the domain name you own, and then press ENTER. Make sure to add the wildcard version to allow your services to use SSL certificates.

<p align="center">
<img src="./images/npm_domain_names.jpg" alt="Adding private domain name to SSL certificate" height=110px>
</p>

Then click the ‘Use a DNS Challenge’ toggle, and then change the DNS Provider to Cloudflare.

<p align="center">
<img src="./images/npm_dns_challenge_setup.jpg" alt="Selecting Cloudflare as DNS provider" height=350px>
</p>

To retrieve your Cloudflare API token for your domain name, log in to Cloudflare, click the profile icon in the top right and select ‘My Profile’.

<p align="center">
<img src="./images/cloudflare_my_profile.jpg" alt="My Profile button in Cloudflare" height=170px>
</p>

Then Click ‘API Tokens’ on the left.

<p align="center">
<img src="./images/cloudflare_api_tokens_tab.jpg" alt="API Tokens tab in Cloudflare" height=250px>
</p>

To the right of ‘API Tokens’, Click ‘Create Token’.

<p align="center">
<img src="./images/cloudflare_create_token.jpg" alt="Creating an API token in Cloudflare" height=120px>
</p>

Click the ‘Edit Zone DNS’ token template.

<p align="center">
<img src="./images/cloudflare_select_token_template.jpg" alt="Selecting the API token template in Cloudflare" height=120px>
</p>

Click ‘Edit Zone DNS’ to change the token name.

<p align="center">
<img src="./images/cloudflare_change_token_name.jpg" alt="Changing the API token name in Cloudflare" height=100px>
</p>

Under ‘Zone Resources’, select your domain name.

<p align="center">
<img src="./images/cloudflare_select_token_zone.jpg" alt="Selecting the API token DNS zone in Cloudflare" height=110px>
</p>

Then click ‘Continue to summary’ > ‘Create Token’ to create your API token.

<p align="center">
<img src="./images/cloudflare_token_summary.jpg" alt="Summarising the API token in Cloudflare" height=220px>
</p>

You will now have an API token for your domain name for issuing SSL certificates. Copy the API token generated and replace the placeholder token with your API token in the ‘Credentials File Content’ section of your NPM SSL certificates window.

<p align="center">
<img src="./images/cloudflare_api_token.jpg" alt="Generated API token in Cloudflare" height=180px>
</p>

<p align="center">
<img src="./images/npm_cloudflare_token_before.jpg" alt="Removing placeholder Cloudflare API token to SSL certificate in NPM" height=150px>
</p>

<p align="center">
<img src="./images/npm_cloudflare_token_after.jpg" alt="Adding Cloudflare API token to SSL certificate in NPM" height=190px>
</p>

Agree to the Let’s Encrypt Terms of Service, then click ‘Save’.

<p align="center">
<img src="./images/npm_ssl_agree.jpg" alt="Agreeing to Terms of Service for SSL certificate" height=140px>
</p>

If you are given this error:

<p align="center">
<img src="./images/npm_internal_error.jpg" alt="Internal Error for SSL certificate creation" height=290px>
</p>

Simply change the propagation seconds to 30-120 seconds, and click 'Save' again.

<p align="center">
<img src="./images/npm_propagation_seconds.jpg" alt="Changing the propagation seconds" height=120px>
</p>

Now you should have a new SSL certificate for your private websites.

<p align="center">
<img src="./images/npm_created_ssl_certificate.jpg" alt="New SSL Certificate for private domain" height=160px>
</p>

---

#### Previous Section: [Setting up Docker on the Raspberry Pi](../docker_setup/)

#### Next Section: [...]()
