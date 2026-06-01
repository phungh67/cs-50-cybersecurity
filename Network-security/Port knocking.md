
>[! Definition]
>**Port knocking** is a clever, "stealth-by-design" network security technique used to protect server ports from being scanned, discovered, or attacked.
>
>It keeps administrative ports (like **SSH on Port 22** or `wireguard VPN` interfaces) completely closed to the public internet by default, only opening them when a client provides a secret, cryptographic-free "secret knock."

### 1. The Core Architecture: Absolute Packet Drops

Normally, if an administrative port is open, a hacker running an `nmap` scan can immediately see it. If it’s closed, the firewall drops it or returns a reset packet.

With Port Knocking, a local firewall (like `iptables` or `nftables`) is configured to **silently drop (`DROP`) all connection attempts to port 22** from the entire internet. To an outside observer or automated bot scanning the network, the port appears completely dead, and the server appears to have no management utilities running at all.

### 2. The Operational Mechanics (The "Secret Knock")

To gain access, a legitimate user must use a specialized port knocking client utility to send a precise sequence of connection attempts to a set of closed, completely arbitrary ports.
		ze
#### Step A: The Knock Sequence

The user's client machine intentionally transmits connection attempts (usually stateless **`UDP`** or **`TCP` `SYN`** packets) to a pre-arranged sequence of numbers.

- _Example Sequence:_ Port 7005 $\rightarrow$ Port 9221 $\rightarrow$ Port 8114 $\rightarrow$ Port 6002.

#### Step B: Passive Daemon Analysis

Behind the firewall sits a specialized, low-overhead background service called a **Port Knocking Daemon** (such as `knockd`). This daemon does not open sockets on those ports; instead, it passively listens to the server's raw kernel packet logs or monitors the interface via `libpcap` (packet capture).
1. The daemon watches the arriving packets from a single source IP.
2. It tracks the sequence: _Did IP `203.0.113.5` hit port 7005? Yes. Did they hit 9221 next? Yes._
3. It checks its local configuration file to see if the sequence matches the secret key.
#### Step C: The Dynamic Firewall Alteration

If the sequence matches exactly, the daemon instantly executes a dynamic system command to temporarily punch a hole in the firewall `ruleset` **specifically for that user's source IP address**:

Bash

```
# The daemon dynamically runs a rule like this behind the scenes:
iptables -A INPUT -s 203.0.113.5 -p tcp --dport 22 -j ACCEPT
```

#### Step D: The Connection Phase

The administrative port 22 is now wide open _only_ for the user's IP address. The user launches their standard SSH client and connects cleanly. After a configured timer expires (e.g., 10 seconds), the daemon removes the firewall rule so no one else can sneak through the opened door, while the user's active, `stateful` SSH connection remains safely established.

### 3. Engineering Limitations & Vulnerabilities

While port knocking is highly effective at eliminating automated log noise and brute-force scanner bots, it introduces severe architectural weaknesses that limit its use in high-availability enterprise environments:

- **Symmetric Replay Attacks:** Classic port knocking is completely unencrypted. If an attacker is passively sniffing traffic on the same network link (a Man-in-the-Middle position), they can see you send packets to ports 7005, 9221, 8114, and 6002. The attacker can simply replay those exact same connection attempts from their own machine to trick the server into opening the firewall door for them. _(Mitigation: Modern variations use **Single Packet Authorization (SPA)**, which wraps an encrypted, time-stamped cryptographic payload inside a single `UDP` packet to prevent replay)._
- **The Single-Point-of-Failure Risk:** If the background `knockd` service crashes or runs out of system memory, the firewall rules will remain permanently closed. Administrators will be completely locked out of their own cloud server infrastructure with no remote recovery path.
- **Network Latency and Packet Reordering:** Because the internet is fundamentally stateless, packets can arrive out of order or experience random packet loss. If your knock sequence is `7005 -> 9221 -> 8114`, but network congestion causes `9221` to arrive _before_ `7005`, the daemon will reject the sequence as an invalid sequence, locking out a legitimate user.

### Summary Checklist for Your Revision

- **The Concept:** Obscuring administrative ports by dropping all traffic by default and dynamically opening a firewall hole only after receiving a precise sequence of connection attempts to closed ports.
- **The Layer:** Operates via **Layer 3/4 header logging** to avoid establishing active transport-layer connections during the knocking phase.
- **The Verdict:** Great for hiding home labs or personal edge servers from internet noise, but dangerous for critical enterprise DevOps infrastructure due to packet reordering issues and replay vulnerabilities.