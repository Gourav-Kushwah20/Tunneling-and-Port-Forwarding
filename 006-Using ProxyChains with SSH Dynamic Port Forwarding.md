# Using ProxyChains with SSH Dynamic Port Forwarding

You can combine **ProxyChains** with **SSH Dynamic Port Forwarding** to securely route application traffic through an SSH tunnel.  
This setup is commonly used for **traffic anonymization**, **bypassing network restrictions**, and **pivoting into internal networks** during penetration testing.

---

## Step 1: Establish an SSH Dynamic Port Forwarding Tunnel

Create a SOCKS proxy using SSH:

```bash
ssh -D 9050 -q -C -N user@remote_host
```

```bash
ssh -D 9050 -q -C -N debain@192.168.1.103
```

```bash
ssh -N -D 127.0.0.1:1080 debain@192.168.1.103
```

### Example

```bash
ssh -D 9050 -q -C -N armour@192.168.1.25
```


## Option Breakdown

* `-D 9050`
  Binds a SOCKS proxy to port **9050** on the local machine

* `-q`
  Suppresses non-critical SSH output

* `-C`
  Enables compression (useful on slow links)

* `-N`
  Does not execute a remote command (tunnel only)

---

## Step 2: Install ProxyChains

If ProxyChains is not already installed, install it using:

```bash
apt install proxychains
```

---

## Step 3: Configure ProxyChains

Edit the ProxyChains configuration file:

```bash
vim /etc/proxychains.conf
```

Add or verify the following SOCKS proxy entry:

```conf
# defaults set to "tor"
socks4 	127.0.0.1 9050
socks4  127.0.0.1 1080
socks5  127.0.0.1 1080
```

---

### Recommended Settings

Ensure the following options are enabled (uncommented):

```conf
strict_chain     # Forces traffic to follow proxies in defined order
proxy_dns        # Resolves DNS queries through the proxy
```

---

## Step 4: Use ProxyChains with the SSH Tunnel

Once the tunnel is active, prepend commands with `proxychains` to route traffic through the SSH SOCKS proxy.


## Examples

### HTTP Request

```bash
proxychains curl http://192.168.2.100
```

### Network Scanning

```bash
proxychains nmap -v -Pn -sT -n -p 80,445,443,3389 192.168.2.100
```

### Browser Traffic

```bash
proxychains firefox
```


