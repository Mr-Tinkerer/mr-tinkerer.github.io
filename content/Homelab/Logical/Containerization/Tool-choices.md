---
title: Tool-choice philosophy
description: The real reason behind running a different Linux distro per VM and picking Podman over Docker — deliberately choosing unfamiliar tools for the learning value, not because they're technically superior.
tags:
  - Homelab
  - Containerization
---
# Why these choices aren't really about "best tool for the job"

The linux distros were picked because besides Debian, I had never used the other ones before for server hosting. I had only used Debian, Ubuntu, and Alpine Linux before. I wanted to learn how to operate new distros, so I made each VM run a different, unique distro. The same underlying reasoning is why I run [[Homelab/Logical/Containerization/Tool-choices#Podman over Docker|Podman]] instead of Docker on every one of those VMs: I'd rather spend the home lab's setup time learning something I haven't used before than default to whatever I already know.

# One distro per VM

Each Platform VM in this home lab runs a different guest OS, on purpose:

| VM              | Guest OS                                                          | Notes                                                                                                           |                                                                                             |
| --------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Productivity VM | [[Homelab/Platform/Productivity-VM/Debian/Debian-overview\|Debian]]      | The one distro of the three I'd actually used before, but paired here with new-to-me services.                  |                                                                                             |
| Personal VM     | [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-gotchas\|openSUSE]]       | First time running a `firewalld`/SELinux-enforcing distro — see [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-gotchas\|openSUSE Gotchas]] for what that cost me. |
| Local AI VM     | [[Homelab/Platform/Local-AI-VM/Fedora/Cpu-configuration\|Fedora]] | First time running an RPM/`dnf`-based distro in this home lab.                                                  |                                                                                             |

See [[Homelab/Logical/Virtual-machines/Vm-allocation|Virtual machines]] for the full VM roster, including the two VMs that don't have services configured yet.

# Podman over Docker

I picked Podman as the container engine for every VM in this home lab instead of Docker, as another instance of the same "learn something new" philosophy above — I'd only ever used Docker before, so running every service on Podman/Quadlet instead was a deliberate choice to learn an unfamiliar tool rather than reach for the one I already knew. It also happens to hold up well on its own technical merits, which made it an easy choice to commit to across the whole home lab:

- **Daemonless.** Docker relies on a long-running root-owned background daemon (`dockerd`) that everything talks to. Podman has no such daemon — each container is a direct child process of whatever invoked it. That's a smaller attack surface and one less thing that can silently fall over and take every container down with it.
- **Rootless-friendly by default.** Podman was built around running containers without root privileges from the start, which matches how I want services on a shared VM to behave — a compromised container process doesn't automatically mean a compromised root daemon.
- **systemd integration via Quadlet.** Since Podman has no daemon to restart containers on boot or on failure the way Docker's daemon does, I needed something else to own that job. [[Homelab/Logical/Containerization/Tool-choices#Podman Quadlet, in practice|Podman Quadlet]] fills that gap: it turns a declarative `.container`/`.network` unit file into a normal systemd service, so every container gets start-on-boot, restart-on-failure, and dependency ordering (`After=`/`Requires=`) for free, using infrastructure (systemd) that's already on every one of my Linux VMs.
- **CLI/API compatibility with Docker.** Podman deliberately mirrors the Docker CLI and supports the same image format, so none of the setup knowledge (Dockerfiles, compose-style thinking, image registries) I'd otherwise use with Docker goes to waste — I just point `docker`-shaped muscle memory at `podman` instead, and quadlets replace `docker-compose.yml`/`docker run` flags with a small ini-style unit file that systemd understands natively.

# Podman Quadlet, in practice

Each service that uses Podman keeps its quadlet definitions in the infrastructure repo (see that service's own Installation/Setup page for the exact file links) — I don't paste maintained quadlet files into the wiki directly, per the external-file-reference rule. A quadlet unit typically defines the image, the network, volume mounts, and port/label metadata; systemd then generates and manages the actual service from it, so `systemctl status <name>` works exactly like it would for any other system service.

# Glossary

See the [[Homelab/Logical/Containerization/Containerization-glossary#Podman|Containerization glossary]] for the canonical definitions of Podman and Podman Quadlet.
