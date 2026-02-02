# SSH Remote Port Forwarding (-R)

### Description : In internal server can't reach external service like `192.168.1.1(router)` so we can use that
Windows Server

SSH Remote Port Forwarding allows a **remote SSH server** to open a listening port and forward incoming connections **back to a service reachable from the SSH client**.

> A port opens on the **remote system**, but the service lives on **your side (or your internal network)**.

---

## When to Use Remote Port Forwarding

- Expose local services to a remote system  
- Bypass NAT / firewall restrictions  
- Reverse pivoting in penetration tests  
- Allow a compromised target to access attacker services  

---

## Syntax

```bash
ssh -R [REMOTE_BIND_ADDR]:[REMOTE_PORT]:[DESTINATION_HOST]:[DESTINATION_PORT] user@SSH_SERVER
```

---

## Parameters Explanation

| Parameter          | Meaning                                            |
| ------------------ | -------------------------------------------------- |
| `REMOTE_BIND_ADDR` | IP to bind on SSH server (`127.0.0.1` recommended) |
| `REMOTE_PORT`      | Port opened on SSH server                          |
| `DESTINATION_HOST` | Host reachable from SSH `client`(attacker side)                     |
| `DESTINATION_PORT` | Service port on destination                        |
| `-N`               | Tunnel only (no shell)                             |

---

## Traffic Flow

```text
Remote Client → SSH Server → Local Service
```


## Basic Example

```bash
ssh -R 9090:localhost:22 root@ssh_server
```

### Explanation: 

Port `9090` opens on `remote_host`

Any Connection to:
```bash
remote_host:90909
```
- Is forwarded to:
```text
localhost:22   (SSH client side)
```

---

## Allow Remote Binding (If You Control the SSH Server)

If you have **root access** to the SSH server and want to allow **remote port forwarding** to bind to a **non-localhost IP**, you must update the SSH daemon configuration.


## Edit SSH Configuration

```bash
vim /etc/ssh/sshd_config
```

### Add or Update the Following Lines

```conf
#AllowAgentForwarding yes
AllowTcpForwarding yes
GatewayPorts yes
```

---

## Restart SSH Service

```bash
systemctl restart ssh
```

---

## Resulting Behavior

* SSH honors the bind address specified with `-R`
* Allows binding to:

  * A specific interface IP (e.g., `192.168.x.x`)
* Requires the IP to **exist on the SSH server**


## Example

```bash
ssh -N -R 192.168.2.103:8888:192.168.1.1:80 armour@192.168.1.103
```


## Expected Listener

```text
192.168.2.103:8888  LISTEN
```
---

## GatewayPorts Modes Explained

### Default – Bind only to `127.0.0.1`
```conf
GatewayPorts no
```

### Bind to Client-Specified IP

```conf
GatewayPorts clientspecified
```

### Bind to `0.0.0.0` (All Interfaces)

```conf
GatewayPorts yes
```

### ⚠️ Warning

`GatewayPorts yes` exposes the forwarded port to **all network interfaces** and is **not recommended** unless explicitly required.

---

```bash
ssh -R 9090:192.168.1.1:80 debainf@192.168.1.103
```
```bash
ssh -R 192.168.2.103:9090:192.168.1.1:80 debainf@192.168.1.103
```
## Explanation

* Remote users connect to `ssh_server:9090`
* Traffic is forwarded to the local SSH service running on port `22`

---

## GatewayPorts Modes Explained

```text
GatewayPorts no                # Default – bind only to 127.0.0.1
GatewayPorts clientspecified   # Bind to client-specified IP
GatewayPorts yes               # Bind to 0.0.0.0 (ALL interfaces)
```

### ⚠️ Warning

`GatewayPorts yes` exposes the forwarded port to **all network interfaces** and is **not recommended** unless explicitly required.

---

## Lab Scenario

### Network Layout

* **Attacker (Kali / SSH Client):** `192.168.1.7`
* **Target (SSH Server):** `192.168.1.103`
* **Attacker's Web Server:** `192.168.1.1:80`

---

## Reverse Pivot – Exposing Internal Web Server

```bash
ssh -N -R 8888:192.168.1.1:80 debain@192.168.1.103
```
```bash
ssh -N -R 192.168.2.103:8888:192.168.1.1:80 debain@192.168.1.103
```

## Result

* Port **8888** opens on the **SSH server** (`192.168.1.103`)


## Accessing

http://192.168.1.103:8888

http://192.168.2.103:8888


## Reaches

```text
192.168.1.1:80
```

- Safe Binding (Recommended)
```bash
ssh -N -R 127.0.0.1:8888:192.168.2.100:80 debain@192.168.1.103
```

---

## Common Use Cases

* Exposing local services behind NAT
* Reverse shells and persistence
* Temporary service sharing

---

## Exposing a Local Web Service to a Remote Host

### HTTP Example (Localhost Binding)

```bash
ssh -N -R 8888:127.0.0.1:80 root@192.168.1.103
```


### HTTP Example (Bind to All Interfaces)

```bash
ssh -N -R 0.0.0.0:8888:127.0.0.1:80 armour@192.168.1.103
```



### Result

* Port **8888** opens on **192.168.1.103**
* Traffic is forwarded to:

```text
localhost:80   (attacker / client side)
```

### HTTPS Example

```bash
ssh -N -R 8443:127.0.0.1:443 root@192.168.1.103
```

---

## ASCII Network Diagram

```text
Remote User
     |
     |  http://127.0.0.1:8888
     |
[ SSH Server ]
  192.168.1.103
     |
     |  Encrypted SSH Tunnel
     |
[ Attacker / Client ]
  192.168.2.100:80
```

---

## Key Points

* Remote Port Forwarding opens a **listener on the SSH server**
* Requests are forwarded **back to the attacker/client machine**
* `0.0.0.0` binding requires `GatewayPorts yes`
* Use `-N` for tunnel-only connections (no shell)


---
## plink (Windows) – Remote Port Forwarding

### Syntax

```bash
plink.exe -N -R [REMOTE_PORT]:[DEST_HOST]:[DEST_PORT] user@SSH_SERVER
```

### Examples

```bash
plink.exe -N -R 80:127.0.0.1:80 root@192.168.1.103
```

```bash
plink.exe -N -R 8080:192.168.200.10:8080 root@192.168.1.103
```

---

## Local vs Remote Port Forwarding

| Feature              | Local (`-L`)    | Remote (`-R`)        |
| -------------------- | --------------- | -------------------- |
| Who accesses service | Attacker        | Remote system        |
| Direction            | Forward         | Reverse              |
| Typical use          | Internal access | Expose local service |
| Exam frequency       | High            | Medium               |

---

## Questions

- What is Local Port forwarding?

- What is Remote Port forwarding? 

- What is Dynamic Port forwarding?
