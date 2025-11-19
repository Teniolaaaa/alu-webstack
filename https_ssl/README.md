```markdown
# HTTPS SSL - alu-webstack (https_ssl)

This folder contains the files needed to configure HAProxy SSL termination and a DNS audit script.

Files
- 0-world_wide_web : DNS audit script (usage: ./0-world_wide_web domain [subdomain])
- 1-haproxy_ssl_termination : HAProxy config to terminate SSL on :443 and proxy to web backends
- 2-redirect_http_to_https : HAProxy config that redirects HTTP -> HTTPS and terminates SSL on :443

Domain and DNS setup (teniola.me)
Create the following A records at your registrar/DNS provider for the domain teniola.me:

- Host/name: www     Type: A   Value: 3.92.68.175    TTL: 300
- Host/name: lb-01   Type: A   Value: 3.92.68.175    TTL: 300
- Host/name: web-01  Type: A   Value: 13.220.103.184  TTL: 300
- Host/name: web-02  Type: A   Value: 3.85.233.7      TTL: 300

Important:
- If you use Cloudflare (or another proxy/CDN), set those records to DNS-only (disable proxying) so they return the real A record IPs rather than Cloudflare IPs.

Obtain TLS certificates (on lb-01)
1) SSH into the load balancer:
   ssh ubuntu@3.92.68.175

2) Install HAProxy and certbot:
   sudo apt-get update
   sudo apt-get install -y haproxy certbot

3) Stop HAProxy temporarily so certbot standalone can use port 80:
   sudo systemctl stop haproxy

4) Request certificate for www.teniola.me:
   sudo certbot certonly --standalone -d www.teniola.me --non-interactive --agree-tos -m t.olaleye@alustudent.com

5) Create the combined PEM for HAProxy (private key first, then fullchain):
   sudo bash -c 'cat /etc/letsencrypt/live/www.teniola.me/privkey.pem /etc/letsencrypt/live/www.teniola.me/fullchain.pem > /etc/letsencrypt/live/www.teniola.me/haproxy.pem'
   sudo chmod 600 /etc/letsencrypt/live/www.teniola.me/haproxy.pem

6) Install HAProxy config:
   # choose either 1-haproxy_ssl_termination (no HTTP->HTTPS redirect) or 2-redirect_http_to_https (redirect enabled)
   sudo cp /path/to/repo/https_ssl/1-haproxy_ssl_termination /etc/haproxy/haproxy.cfg
   # or
   sudo cp /path/to/repo/https_ssl/2-redirect_http_to_https /etc/haproxy/haproxy.cfg

7) Start and enable HAProxy:
   sudo systemctl enable --now haproxy

8) Verify:
   curl -Ik https://www.teniola.me
   curl -s https://www.teniola.me

Auto-renewal
- Test renewal:
  sudo certbot renew --dry-run
- Ensure HAProxy is reloaded when certs renew:
  sudo bash -c 'echo "systemctl reload haproxy" > /etc/letsencrypt/renewal-hooks/deploy/reload-haproxy.sh && sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-haproxy.sh'

DNS & verification commands (local machine)
- Check DNS propagation:
  dig +short www.teniola.me @8.8.8.8
  dig +short web-01.teniola.me @8.8.8.8
  dig +short web-02.teniola.me @8.8.8.8

- Run audit script:
  ./0-world_wide_web teniola.me
  ./0-world_wide_web teniola.me www
```
