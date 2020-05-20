# Ansible Pi-hole

Basic Pi-hole configuration for my Pi-hole. I shared it with you, even though it's made for me.

## DNS-over-HTTPS

`cloudflared` works to proxy to Families at 1.1.1.3 and 1.0.0.3 instead of the defaults. Pi2 with armhf doesn't work with the arm6 deb installers provided by Cloudflare - I had to do the manual binary install.

* https://developers.cloudflare.com/argo-tunnel/downloads/
* https://docs.pi-hole.net/guides/dns-over-https/
