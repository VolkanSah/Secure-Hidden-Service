# Setting Up a Secure Hidden Service (Onion Site) - basics

## Table of Contents
1. [Introduction](#introduction)
2. [Why Avoid Exit Traffic?](#why-avoid-exit-traffic)
3. [Installing and Configuring Tor](#installing-and-configuring-tor)
4. [Setting Up a Hidden Service](#setting-up-a-hidden-service)
5. [Firewall and Network Security](#firewall-and-network-security)
6. [Disabling Logging for Anonymity](#disabling-logging-for-anonymity)
7. [Securing the Server](#securing-the-server)
8. [Conclusion](#conclusion)

# Other important stuff
- [Use mltiple Tor Instances for Hidden Services/Tunnels](https://github.com/VolkanSah/run-multiple-Tor-instances)
- [ModSecurity Webserver Protection Guide](https://github.com/VolkanSah/ModSecurity-Webserver-Protection-Guide)
- [ModSecurity Rule to Block SQL Injection Attacks in PHP](https://github.com/VolkanSah/ModSecurity-rule-to-block-SQL-injection-attacks-in-PHP)

## Introduction
A Tor Hidden Service (onion site) allows you to host a website that is only accessible via the Tor network. To maintain security and anonymity, we must ensure that our service does not act as an exit node, which could expose our server to legal and security risks.

## Why Avoid Exit Traffic?
Exit nodes in the Tor network route traffic from users to the regular internet, potentially exposing the server operator to liability. To prevent this, we configure our Tor service to only act as a hidden service without relaying or exiting traffic.

## Installing and Configuring Tor

### Install Tor
```bash
sudo apt update && sudo apt install tor -y
```

### Configure Tor to Disable Exit Traffic
Edit the Tor configuration file (`/etc/tor/torrc`) and add the following lines:
```ini
SocksPort 0  # Disable Socks proxy
ExitRelay 0  # Prevent exit traffic
```
Restart Tor to apply the changes:
```bash
sudo systemctl restart tor
```

## Setting Up a Hidden Service
Modify the `torrc` file to configure your hidden service:
```ini
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
```
Create the directory and set proper permissions:
```bash
sudo mkdir -p /var/lib/tor/hidden_service/
sudo chmod 700 /var/lib/tor/hidden_service/
sudo chown -R debian-tor:debian-tor /var/lib/tor/hidden_service/
```
Restart Tor:
```bash
sudo systemctl restart tor
```
Retrieve your new `.onion` address:
```bash
cat /var/lib/tor/hidden_service/hostname
```

## Firewall and Network Security
To ensure your server only listens on local connections:
```bash
sudo ufw allow 22/tcp  # SSH access (if needed)
sudo ufw allow 80/tcp  # Web traffic via Tor
sudo ufw enable
```
For Nginx, modify the configuration to bind only to localhost:
```nginx
server {
    listen 127.0.0.1:80;
    server_name _;
    root /var/www/html;
}
```
For Apache:
```apache
<VirtualHost 127.0.0.1:80>
    DocumentRoot "/var/www/html"
</VirtualHost>
```

## Disabling Logging for Anonymity
To prevent information leaks, disable logging:

### Nginx
```nginx
access_log off;
error_log /dev/null crit;
```

### Apache
```apache
CustomLog /dev/null common
ErrorLog /dev/null
```

## Securing the Server
- **Use Fail2Ban** to prevent brute-force attacks:
```bash
sudo apt install fail2ban -y
```
- **Secure SSH:**
  - Disable root login (`PermitRootLogin no` in `/etc/ssh/sshd_config`)
  - Change default SSH port (`Port 2222` instead of `Port 22`)
- **Encrypt sensitive data** using LUKS or encfs.

## Conclusion
By following these steps, you will have a secure and anonymous Tor Hidden Service without exposing your server to exit traffic risks. Maintain best security practices and keep your software updated to protect your service. 🚀

## Copyright
S. Volkan Sah
