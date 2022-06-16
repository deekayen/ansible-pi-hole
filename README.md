# Ansible Pi-hole

Basic Pi-hole configuration for my Pi-hole. I shared it with you, even though it's made for me.

## DNS-over-HTTPS

`cloudflared` works to proxy to Families at 1.1.1.3 and 1.0.0.3 instead of the defaults. Pi2 with armhf doesn't work with the arm6 deb installers provided by Cloudflare - I had to do the manual binary install.

* https://developers.cloudflare.com/argo-tunnel/downloads/
* https://docs.pi-hole.net/guides/dns-over-https/

## log2ram

https://github.com/azlux/log2ram

```
SIZE=100M
USE_RSYNC=false
MAIL=true
PATH_DISK="/var/log"
ZL2R=false
COMP_ALG=lz4
LOG_DISK_SIZE=100M
```

Configure the journal to stay small, otherwise it'll lock-up the system when log2ram runs out of space, then the log2ram service won't start again.

```
journalctl --vacuum-size=8M
```
## Disable rate limit

Specific rate limit can be defined in /etc/pihole/pihole-FTL.conf configuration file. By default rate limit is set to allow no more than 1000 queries per 60 seconds like `RATE_LIMIT=1000/60`. Fix it with:

```
RATE_LIMIT=0/0
```
