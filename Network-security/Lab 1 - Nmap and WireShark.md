---
title: 1. Scenario

---


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
## Question 1
>Why are higher port numbers (>1024) still of interest for hackers or penetrations testers?

The lower ports, while they are critical to the system (port 53, 22, 25,...) but they are managed by the OS and are also frequently audited, so attacking these targets requires many efforts and maybe the result is useless.

In the other hand, the higher ports are not only less secure, but also come from some important service, notably `mysql` in 3306 and `postgresql` in 5432. Also, some higher ephemeral ports are less likely protected appropriately in the firewall rules, so these ports are the more feasible targets than the lower ones.

## Question 2
>Resulting packets in different scanning scenarios, inspect them by using `Wireshark`.

Commands that we used for this question
```bash
# scan the virtual interface of windows host
sudo nmap -v -sS -p 1-1024 10.0.2.2
sudo nmap -v -sA -p 1-1024 10.0.2.2
sudo nmap -v -sF -p 1-1024 10.0.2.2

# scan the physical interface, the ip is 129.16.22.18
sudo nmap -v -sS -p 1-1024 129.16.22.18
sudo nmap -v -sA -p 1-1024 129.16.22.18
sudo nmap -v -sF -p 1-1024 129.16.22.18

# scan the scanning target with firewall disable/enable
sudo nmap -v -sS -p 1-100 10.0.0.2
sudo nmap -v -sA -p 1-100 10.0.0.2
sudo nmap -v -sF -p 1-100 10.0.0.2
```

`-sS` is the stealth scan, only send the `SYN` then reset immediately. This allows the result to be returned nearly immediately, saving time
`-sT` is the scan with the `connection()` calls,  use when do not have root privilege, establishes the full 3-ways handshake.
`-sA`, only send the `ACK` then `RST` to probe the firewall rule (to check if the returned results were filtered or not).
`-sW` to check the window size in `RST`.
`-sX` (`-sN`) Xmas and Null.

## Question 3
> Some scans take a long time to complete. Why?! Look at the responses from the system and try to explain! (The answer has been mentioned during a lecture.)

The slow scans are mainly caused by missing responses. If the target replies with `RST` or `SYN-ACK`, `nmap` can decide quickly. But if a firewall drops packets silently, `nmap` has to wait for timeout and retry, which makes the scan slow.

The main reasons are `timeout` and `retransmission`.

When `nmap` sends a probe (like a `SYN`), it expects one of two things immediately: a `SYN/ACK` (Open) or an `RST` (Closed).

- If the target is up and does not have any firewall, `nmap` tends to return immediately, and moves to next ports (in the input list).
- If the target has firewall that drops suspicious packets, these will be dropped, but `nmap` cannot really know this, it must wait for the time out, then re sends the packet, increases the waiting time.

## Question 4
> Nmap can report ports to be in dierent states. What do the states ”open”, ”closed”, ”filtered” and ”unfiltered” mean? Read the manual page and documentation! Which are the most/least interesting ones? Run a scan against the target systems and try to find examples of all these types of ports. Look at the responses from the system. Can you match the replies with the di↵erent states?

- **Open**: An application is actively listening for connections.`SYN/ACK`
- **Closed**: No application is listening, but the host is reachable.`RST` (Reset)
- **Filtered**: A firewall/filter is blocking the probes. Packet was dropped, so cannot tell if it closed or blocked (no response).
- **Unfiltered**: The port is accessible, but `nmap` can't determine if it's open or closed.

Most interested is **Open**, actively has service running on it and does not have any protection mechanism, this is the sweet spot to conduct attacks.
## Question 5
> What are the similarities and differences between the five scan types, i.e., syn, ack, fin, null, and Xmas? What are their use-cases, i.e., in which situation can each scan be useful?

```bash
sudo nmap -v -sN -p 22,80,443 10.0.2.2
sudo nmap -v -sX -p 22,80,443 10.0.2.2
```
Null scan sends a TCP packet with no flags set. Xmas scan sends a TCP packet with FIN, PSH, and URG set. In Wireshark, these flags can be seen in the TCP header. Closed ports often reply with RST, while open or filtered ports often give no response.

### Their similarity
All five scan types will send specially crafted packets to the target port. And using the reply result to deduce the port status. All of them are using half-open or abnormal-flag probing, in order to avoid making a full TCP connection.

### Their differences
1. SYN scan: It only sends a packet with the SYN flag set. It is useful when we want to find which TCP ports are open.
2. ACK scan: it sends a packet with the ACK flag set. It is used to see whether a firewall lets a packet through.
3. FIN scan: it sends a packet with only the FIN flag set. It can be useful when a firewall blocks the SYN packet.
4. NULL scan: it sends a TCP packet with no flags set at all. It is an alternative to the FIN scan.
5. Xmas scan: it sends a packet with FIN + PSH + URG set. It is very similar to the FIN/NULL scan.

## Question 6 
> How is UDP scanning done by Nmap? Why is this type of scan more problematic than TCP scans?

```bash
sudo nmap -v -sU -p 1-1024 10.0.0.2
```

UDP scanning is done by sending UDP probes and checking whether the target replies. Closed ports usually return ICMP port unreachable, but open ports often give no response. That makes UDP scanning harder than TCP scanning, because there is no handshake and no reply can mean either open or filtered. Also, ICMP replies are often rate-limited, so UDP scans are slow.

First, `nmap` sent a simple and empty packet for some unknown ports. But for several known port (for example, the port 53), `nmap` will send protocol-specific payloads, trying to get a response. And because UDP is a connectionless protocol, many packets would not be returned.

This problematic because it really takes a lot of time to scan the UDP, although the sending was fast, because it lacks the handshake, but it requires time to wait for a response, since maybe many retransmissions maybe needed to determine this port is opened or not.
## Question 7
>How much time does it take to complete a UDP scan on the system. Is this time varies between different systems? Explain?

```bash
# we use time command to check the actual time
time sudo nmap -v -sU -p 1-1024 10.0.2.2        # it costs 201.93s
time sudo nmap -v -sU -p 1-1024 129.16.22.18    # it costs 402.03s
time sudo nmap -v -sU -p 1-1024 10.0.0.2        # disable firewall, it costs 1104.80s
time sudo nmap -v -sU -p 1-1024 10.0.0.2        # enable firewall, it costs 14.22s
```

Some system has "based-rate" limited, so when the `nmap` hit that rate, it must wait and try again, so it takes a lot of time to complete a full UDP scan. Typically, it take roughly 1 second to scan a port, and if we scan whole system with 65353 ports, it will take about 18 hours. But for the first 1000 ports, ~maybe 49 minutes. Tried with the target-scanning VM, 

Different by systems:
- Linux stops responding if the limit was hit, this creates the wait-and-try loop, quite takes time.
- Windows, allows a burst at start, but otherwise, slowing down after the limit was hit. So, fast at start, slow gradually (haven't tested yet)
- If there is a firewall, that `DROP` instead of `REJECT`, `nmap` must wait for the time out, typically, more than 1 second, so this is the slowest scenario so far.
- And if the system does not have any filter, the scan is fast, extremely fast.

## Question 8
> What packet(s) does Nmap send to figure this out? Also, look at the responses sent by the systems using Wireshark. Do you see any differences that may reveal what type of system is being used? (You may want to limit the number of ports it scans to avoid waiting, and look at protocol parameters. Which ports should you select?!)

```bash
# in order to speed up, we can use specify ports, one open port and one closed port
sudo namp -O -p 80,81 10.0.2.2
```
`nmap` uses the flag `-O` to detect the operating system of a machine. It use multiple crafted packets rather than just one packet. It sends TCP, ICMP, and UDP packets to the target, including packets to a known open TCP port, a known closed TCP port, and a closed UDP port (if possible). And `nmap` will compare the replies with its fingerprint database.

Different operating systems have different reply fields, such as TTL, TCP window size, and ICMP unreachable formatting. `nmap` could use that information to analyze the operating system.

For OS detection, it is best to select at least one known open TCP port and one known closed TCP port, because `nmap` uses the differences in the responses to both types of ports to build a more reliable fingerprint.

## Question 9
> Why would you want to use fragmented packets for scanning? How do you think fragmented packets should be handled in modern networks?

Fragmentation can be used to evade simple filters or IDS/IPS that do not properly reassemble packets. Modern networks should inspect or reassemble fragments carefully and drop suspicious ones.

Fragmented packets can be used in scanning to test whether a firewall or IDS can properly inspect fragmented traffic, and in some cases to evade simple filters that only inspect the first fragment. In modern networks, fragments should be reassembled or normalized before filtering, and suspicious tiny or overlapping fragments should usually be dropped. However, fragmented traffic can also be legitimate, so networks should not blindly block everything without considering real cases.

## Question 10
>When checking the output with Wireshark, do you see fragmented packets as expected? What happens if you add the --send-eth option?

```bash
sudo nmap -sS -f -p 80 10.0.0.2
sudo nmap -sS -f --send-eth -p 80 10.0.0.2

# we can use the following filter to find the fragmented IP protocol packets
ip.addr == 10.0.0.2 && (ip.flags.mf == 1 || ip.frag_offset > 0 || tcp)
```

-f makes `nmap` send fragmented IP packets. In `Wireshark`, we expect to see multiple fragments. `--send-eth` makes `nmap` send at Ethernet level, so the OS interferes less, and the fragmentation behavior may become clearer or more reliable.

In `Wireshark`, we did not always see fragmented packets with `-f` alone, because the sending OS may alter or reassemble them before they go out. Adding `--send-eth` bypasses the IP layer and usually makes the fragmentation visible as expected.

## Question 11
> If you are going to remember only one thing from this lab, what should it be? (There is no right answer here)

For me, the one thing that I could remember is that a firewall is super important for an operating system, since an attacker could get a lot of information about my machine by using a small tool.

# My feedback
1. We need to log our commands when we do the testing. Yep, and the most important flag, in my opinion was `-Pn`, if not, `nmap` would assume the destination was dead, hence returned useless response.  
2. In order to speed up the scanning, we could set the port range to 1-1024, and should use the stealth scan `-sS`.
3. We need to do four tests for each step.
4. For questions 2 and 4, do we need to paste some scanning results for those questions?, I doubt that, but tomorrow, we can scan again, then show the TAs.
5. For question 7: Maybe we need to do the test in the virtual machine, rather than just give an approximate time. Already test. A full scan take 48 minutes (first 1024 ports) and 3 hours 49 minutes if full ports.
	1. For question 8, do we need to run the command in the virtual machine and paste the result for Windows and Linux? I really don't know if can scan Window, since the VMs are isolated?