# `sshuttle` – Transparent Proxy / VPN over SSH

* **Repo:** [https://github.com/sshuttle/sshuttle](https://github.com/sshuttle/sshuttle)
* **Requirement:** You must run as root or via sudo (iptables / firewall rules).

---

## Lab Setup

* **Target (Jump / Pivot Host):** `192.168.1.103`
* **Attacker:** `192.168.1.7`

---

## Installation

```bash
apt update
```
```
apt install sshuttle
```

### Check help

```bash
sshuttle --help
```

---

## Basic Connection (No Traffic Routed)

*Just verifies SSH connectivity*

```bash
sshuttle -vNHr root@192.168.1.201
```

### Flags

* `-v` : verbose
* `-N` : no subnet yet
* `-H` : auto hosts file handling
* `-r` : remote ssh hosts

---

## 🚇 Tunnel ALL Traffic (Full VPN-like)

```bash
sshuttle -vr root@192.168.1.103 0.0.0.0/0
```

> Routes all IPv4 traffic via the remote host
> This will route everything, including internet traffic.

---

## 🚫 Exclude Specific Traffic

### Exclude a single IP:

```bash
sshuttle -vr root@192.168.1.103 0.0.0.0/0 -x 192.168.1.21
```

### Exclude a subnet:

```bash
sshuttle -vr root@192.168.1.103 0.0.0.0/0 -x 192.168.1.0/24
```

---

## 🎯 Tunnel Specific Subnets (Recommended for PT)

### Single subnet:

```bash
sshuttle -vr root@192.168.1.103 192.168.2.0/24
```

### Multiple subnets:

```bash
sshuttle -vr root@192.168.1.103 192.168.2.0/24 192.168.100.0/24 192.168.200.0/24
```

---

## ⚙️ Daemon Mode + SSH Key

```bash
sshuttle -D -r root@192.168.1.103 192.168.200.0/24 --ssh-cmd "ssh -i ./id_rsa"
```

* `-D` → run in background (daemon)

---

## 🌐 Forward DNS Queries

```bash
sshuttle --dns -vr root@192.168.1.103 192.168.2.0/24
```

### Verify:

```bash
curl http://192.168.2.100
```

---

## 🧠 DNS Only + No Subnet (Rare Case)

```bash
sshuttle --dns -vNhr root@192.168.1.201
```

> Useful when you only need DNS resolution through the remote host
> (no traffic tunneling).

---

## 🔐 Sudoers Handling (No sudoers file modification)

```bash
sshuttle --sudoers-no-modify --sudoers-user armour -r root@192.168.1.201 192.168.200.0/24
```

### Use this when:

* You don’t want **sshuttle** to touch `/etc/sudoers`
* You already have sudo rights

---

## 🧪 Best-Practice Notes (PT / Red Team)

* Prefer **specific subnets**, not `0.0.0.0/0`
* Always test with:

  * `curl`
  * `ping`
  * `ip route`
* Combine with:

  * `proxychains` (for non-TCP tools)
  * `nmap -Pn` (avoid ICMP dependency)
* If DNS fails → **always add `--dns`**

---
