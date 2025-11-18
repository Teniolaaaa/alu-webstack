# HTTPS SSL - alu-webstack (https_ssl)

This directory contains work for the "HTTPS SSL" project: a small Bash DNS-audit script and example HAProxy configuration files to demonstrate SSL termination and HTTP->HTTPS redirection.

Prerequisites
- Ubuntu 16.04 LTS (the project tests run on this environment)
- bash (scripts require `#!/usr/bin/env bash`)
- dig (from bind9-dnsutils)
- awk
- haproxy >= 1.5
- certbot (letsencrypt) to obtain certificates
- root privileges to place HAProxy configuration and certificate files

Files
- 0-world_wide_web
  - A Bash script that audits DNS records for specific subdomains.
  - Usage: ./0-world_wide_web domain [subdomain]
    - When only `domain` is given, it reports www, lb-01, web-01 and web-02 (in that order).
    - When `domain` and `subdomain` are given, it reports only the given subdomain.
  - Outputs lines like:
    The subdomain web-02 is a A record and points to 54.89.38.100
  - Implementation notes:
    - Uses `dig` and `awk`
    - Contains at least one Bash function
    - Script is executable and follows the required shebang and comment line
    - Shellcheck-compatible (no errors with Shellcheck v0.3.7)

- 1-haproxy_ssl_termination
  - Example HAProxy configuration demonstrating SSL termination on port 443.
  - Replace certificate path with your cert+key PEM file for your domain.
  - HAProxy must be listening on TCP 443 and must serve proxied backend content.

- 2-redirect_http_to_https
  - Example HAProxy configuration that returns a 301 redirect for HTTP traffic and terminates SSL on 443.
  - Combines the redirect behavior and the SSL frontend/backend setup.

How to obtain and install certificates for HAProxy
1. Obtain certificates with certbot on the load-balancer for your `www` subdomain:
   sudo certbot certonly --standalone -d www.example.com
2. Create the PEM file that HAProxy needs (private key first, then full chain):
   sudo cat /etc/letsencrypt/live/www.example.com/privkey.pem /etc/letsencrypt/live/www.example.com/fullchain.pem > /etc/letsencrypt/live/www.example.com/haproxy.pem
   Ensure the combined file is readable by the haproxy user (or root if haproxy runs as root).
3. Put the chosen HAProxy configuration into `/etc/haproxy/haproxy.cfg` (or include it) and restart/reload HAProxy:
   sudo systemctl restart haproxy

DNS setup
- Add the following A records in your DNS zone:
  - www -> load-balancer IP (lb-01)
  - lb-01 -> load-balancer IP
  - web-01 -> web-01 IP
  - web-02 -> web-02 IP

Notes and tips
- HAProxy must be version 1.5 or higher to support SSL termination features used here.
- The example HAProxy configs reference `/etc/letsencrypt/live/www.example.com/haproxy.pem`. Update the domain in the path to match your certificate location.
- The audit script intentionally ignores advanced edge cases as per project requirements (empty parameters, non-existent domains).
- All files should end with a newline.

Contact / next steps
- Use the script to verify DNS, request certs on the load-balancer, create the combined PEM, drop the HAProxy config into `/etc/haproxy/haproxy.cfg` and restart haproxy to enable SSL termination and HTTP->HTTPS redirection.
