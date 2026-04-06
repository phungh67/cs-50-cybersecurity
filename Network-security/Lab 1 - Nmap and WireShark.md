
# 1. Scenario

The intuition is simple, we have an in-between machine, runs `kali-linux` to inspect 2 other machines: the host machine and a dummy machine that is in the same internal network with that `kali-linux`.

Simplify, we have 2 groups:
- Group 1: the host machine and the `kali` machine, because the `kali` machine has NAT attached, which allows it to not-so-directly talk with underlying host, but still be able to scan host machine.
- Group 2: the `kali` machine and the dummy machine, which are all in a customized network.

# 2. Machine details and the reason behind

Instruction showed that, to scan the virtual interface of the host machine, from `kali` machine, using the IP address  `10.0.2.2`, but why?
- First, because `kali` and the host are linked through a virtual NAT - that is managed by the Oracle Virtual Box itself, so the IP address will have the from of `10.0.x.x`.
- Typically, the address `10.0.2.1` is for the virtual Gateway, `10.0.0.2` is for the host loop back (local host) - in this case, is the underlying machine virtual interface's IP address.
- `10.0.2.3` is for the DNS server and maybe `10.0.2.15` (in most cases, this is the first "empty" address and peferrably given to the virtual machine).

# 3. Scanning result
## 3.1. Scanning host machine

Host machine requires 2 separate scans: one for the virtual network (which can be done via the address `10.0.2.2`) and the other is the physical network interface (inspect the main network interface of the system then scan).

```bash
sudo nmap -sS -Pn -v -p- -sV 10.0.2.2
```

In this command, we have these options:
- `-sS` for `SYN` scan, only get the `SYN`, not wait for full 3-ways `TCP` handshake.
- `-Pn` treated host as up (since several hosts have their firewall denied the `ping` method).
- `-v` for verbose.
- `-p-` to scan all ports, default is first 1000 ports.
- `-sV` to also check the services name.

Scanning result:
```bash
# virtual interface
PORT      STATE SERVICE    VERSION
1716/tcp  open  tcpwrapped
5355/tcp  open  llmnr?
34463/tcp open  unknown

# physical interface
PORT      STATE SERVICE    VERSION
1716/tcp  open  tcpwrapped
5355/tcp  open  llmnr?
```

The `tcpwrapped` is a security tool for `GNU` based machine. While `llmnr` is Link-local Multicast Name Resolution, Virtual Box uses it for naming resolution.

And the different is only `34464` - a very high number port, is because this is an ephemeral process - the `guest-addition-cd`, a tool for shared clipboards, shared directories between host and virtual machines.

It is only visible via virtual interface, not physical interface, because this service only runs in the virtual machine side, not the host's side.

## 3.2. Inspect packets with WireShark


# 4. Question
>[!Question 1]
>Why are higher port numbers (>1024) still of interest for hackers or penetrations testers?

The lower ports, while they are critical to the system (port 53, 22, 25,...) but they are managed by the OS and are also frequently audited, so attacking these targets requires many efforts and maybe the result is useless.

In the other hand, the higher ports are not only less secure, but also come from some important service, notably `mysql` in 3306 and `postgresql` in 5432. Also, some higher ephemeral ports are less likely protected appropriately in the firewall rules, so these ports are the more feasible targets than the lower ones.

>[!Question 2] 
>Resulting packets in different scanning scenarios, inspect them by using `Wireshark`.

`-sS` is the stealth scan, only send the `SYN` then reset immediately. This allows the result to be returned nearly immediately, saving time
`-sT` is the scan with the `connection()` calls,  use when do not have root privilege, establishes the full 3-ways handshake.
`-sA`, only send the `ACK` then `RST` to probe the firewall rule (to check if the returned results were filtered or not).
`-sW` to check the window size in RST.
`-sX` (`-sN`) Xmas and Null.

>[!Question 3] 
>Why do some scans take so long?

The short answer (likely from your lecture) is **Timeouts** and **Retransmissions**.

When Nmap sends a probe (like a `SYN`), it expects one of two things immediately: a `SYN/ACK` (Open) or an `RST` (Closed).

- **The "Fast" Scenario:** If the target is alive and has no firewall, it responds instantly. Nmap moves to the next port in milliseconds.
    
- **The "Slow" Scenario:** If a firewall is set to **DROP** packets (Filtered), the target sends **nothing** back. Nmap doesn't know if the packet was lost in transit or if the host is just slow.
    
- **The Penalty:** Nmap must wait for a "Timeout" (often 1–2 seconds) before giving up. To be sure, it usually **retransmits** the probe. If you scan 1,000 ports and each one requires a 2-second wait plus a retry, a scan that should take 5 seconds suddenly takes 40 minutes.

>[!Question 4] 
>State of `nmap`

StateTechnical MeaningPacket Response (Wireshark)
**Open** An application is actively listening for connections.`SYN/ACK`
**Closed**No application is listening, but the host is reachable.`RST` (Reset)
**Filtered**A firewall/filter is blocking the probes. Nmap can't tell if the port is open or closed.**No response** (or ICMP admin prohibited)
**Unfiltered**The port is accessible, but Nmap can't determine if it's open or closed (usually from an ACK scan).`RST`

Most interested is **Open**, actively has service running on it and does not have any protection mechanism.

>[! Question 6] 
>How the UPD scanning is done by nmap and why is this kind more problematic?

First, `nmap` sent a simple and empty packet for some unknown ports. But for several known port (for example, the port 53), `nmap` will send protocol-specific payloads, trying to get a response. And because UDP is a connectionless protocol, many packets would not be returned.

This problematic because it really takes a lot of time to scan the UDP, although the sending was fast, because it lacks the handshake, but it requires time to wait for a response, since maybe many retransmissions maybe needed to determine this port is opened or not.

=="UDP scanning is inherently more difficult because UDP is connectionless and provides no 'handshake' for confirmation. Nmap identifies **closed** ports by listening for **ICMP Port Unreachable** messages. However, many systems implement **ICMP Rate Limiting**, forcing Nmap to slow down significantly to avoid missing responses. Furthermore, because open UDP services often only respond to valid protocol-specific payloads, Nmap must frequently label non-responsive ports as `open|filtered`, leading to ambiguity that requires time-consuming re-transmissions to resolve."==

>[!Question 7]
>How much time does it take to complete a UDP scan on the system. Is this time varies between different systems? Explain?

Yes, as mentioned before, some system has "based-rate" limited, so when the `nmap` hit that rate, it must wait and try again, so it takes a lot of time to complete a full UDP scan. Typically, it take roughly 1 second to scan a port, and if we scan whole system with 65353 ports, it will take about 18 hours. But for the first 1000 ports, maybe 49 minutes.

Different by systems, yes. For example"
- Linux stops responding if the limit was hit, this creates the wait-and-try loop, quite takes time.
- Windows, allows a burst at start, but otherwise, slowing down after the limit was hit. So, fast at start, slow gradually.
- If there is a firewall, that `DROP` instead of `REJECT`, `nmap` must wait for the time out, typically, more than 1 second, so this is the slowest scenario so far.
- And if the system does not have any filter, the scan is fast, extremely fast.
