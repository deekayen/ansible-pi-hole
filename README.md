# ansible-pi-hole

[![CI](https://github.com/deekayen/ansible-pi-hole/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-pi-hole/actions/workflows/ci.yml) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue)

A personal Ansible playbook that adds DNS-over-HTTPS to an existing [Pi-hole](https://pi-hole.net/) on a Raspberry Pi. It is published as a working example, not a general-purpose role, so the inventory and settings match one home network.

The playbook does not install Pi-hole. It installs [cloudflared](https://github.com/cloudflare/cloudflared) as a local DNS-over-HTTPS proxy on port 5053 for Pi-hole to use as its upstream, following the [Pi-hole DoH guide](https://docs.pi-hole.net/guides/dns/cloudflared/).

## Requirements

- Ansible on the controller. CI installs the current `ansible` package from PyPI.
- A Raspberry Pi with Pi-hole already installed, reachable over SSH, with sudo rights for `ansible_user`.
- A 32-bit ARM (`armhf`) userland. The playbook downloads the `linux-arm` cloudflared build.

## Usage

1. Edit `inventory` so the `pihole` group points at your host and `ansible_user` matches its login.
2. Run the playbook:

   ```bash
   ansible-playbook -i inventory pi-hole.yml
   ```

3. In the Pi-hole admin UI, set the custom upstream DNS server to `127.0.0.1#5053`.

## Playbook tasks

`pi-hole.yml` runs against the `pihole` group with `become: true` and does the following:

| Task | Result |
| --- | --- |
| Install `net-tools` | Provides `netstat` for troubleshooting. |
| Download cloudflared | Unpacks the binary to `/usr/local/bin/cloudflared`; skipped if it already exists. |
| Create the `cloudflared` user | System account with no home directory and `/usr/sbin/nologin`. |
| Copy `files/etc/cloudflared/config.yml` | `proxy-dns` on port 5053, upstreams `family.cloudflare-dns.com` and `doh.familyshield.opendns.com`. |
| Copy `files/etc/default/cloudflared` | `CLOUDFLARED_OPTS` with port 5053 and upstreams `1.1.1.3` and `1.0.0.3` (Cloudflare for Families). |
| Run `cloudflared service install` | Creates `/etc/systemd/system/cloudflared.service`, then enables and starts it. |

Changes to either config file restart `cloudflared`.

## Inventory

| Group | Host | Variables |
| --- | --- | --- |
| `pihole` | `192.168.0.96` | `ansible_user=ubuntu` |

## Known issues

- The cloudflared download URL, `https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.tgz`, returned HTTP 404 when checked on 2026-10-02. On a host without `/usr/local/bin/cloudflared`, that task fails. Current builds are published on the [cloudflared releases page](https://github.com/cloudflare/cloudflared/releases).
- `files/etc/dnsmasq.d/99-edns.conf` (`edns-packet-max=1232`) is in the repository, but no task copies it. Install it by hand if you want it.
- The `Reboot` handler is defined but no task notifies it.

## Manual settings

These are applied by hand on the Pi-hole host, outside the playbook.

### Rate limit

Pi-hole FTL allows 1000 queries per 60 seconds by default (`RATE_LIMIT=1000/60`). To turn the limit off, set this in `/etc/pihole/pihole-FTL.conf`:

```ini
RATE_LIMIT=0/0
```

### log2ram

An earlier version of this setup used [log2ram](https://github.com/azlux/log2ram). It was removed in July 2021 (commit `fe88dc5`). When its RAM disk ran out of space the system locked up, and the log2ram service would not start again.

## Development

CI runs `ansible-lint` on every push to `main` and every pull request (see `.github/workflows/ci.yml`). Locally:

```bash
pip3 install ansible ansible-lint
ansible-lint
```

`.pre-commit-config.yaml` runs the same `ansible-lint` hook; install it with `pre-commit install`.

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Author

[David Norman](https://github.com/deekayen). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
