# Intro to Network Traffic Analysis

Use case of NTA:
- Collecting real time traffic
- Setting a baseline for day-to-day network communication
- Identify and analyzing traffic
- Detecting malware on the wire

NTA workflow
![](https://academy.hackthebox.com/storage/modules/81/workflow.png)

# Tcpdump

List all available interfaces
```sh
sudo tcpdump -D
```

Choosing interface to capture from + START
```sh
sudo tcpdump -i eth0
```

Display name resolution
```sh
sudo tcpdump -i eth0 -nn
```

Display ethernet header
```sh
sudo tcpdump -i eth0 -e
```

Include ASCII and Hex output
```sh
sudo tcpdump -i eth0 -X
```

Switch combinations
```sh
sudo tcpdump -i eth0 -nnvXX
```

Save output to file
```sh
sudo tcpdump -i eth0 -w ~/output.pcap
```

Read output from file
```sh
sudo tcpdump -r ~/output.pcap

# We can disable hostname and port resolution for simplicity and ensure we see any TCP sequence and acknowledgment numbers in absolute values
tcpdump -nnSXr [file/to/read.pcap]
```

#### Helpful TCPDump Filters

| **Filter**           | **Result**                                                                                               |
| -------------------- | -------------------------------------------------------------------------------------------------------- |
| host                 | `host` will filter visible traffic to show anything involving the designated host. Bi-directional        |
| src / dest           | `src` and `dest` are modifiers. We can use them to designate a source or destination host or port.       |
| net                  | `net` will show us any traffic sourcing from or destined to the network designated. It uses / notation.  |
| proto                | will filter for a specific protocol type. (ether, TCP, UDP, and ICMP as examples)                        |
| port                 | `port` is bi-directional. It will show any traffic with the specified port as the source or destination. |
| portrange            | `portrange` allows us to specify a range of ports. (0-1024)                                              |
| less / greater "< >" | `less` and `greater` can be used to look for a packet or protocol option of a specific size.             |
| and / &&             | `and` `&&` can be used to concatenate two different filters together. for example, src host AND port.    |
| or                   | `or` allows for a match on either of two conditions. It does not have to meet both. It can be tricky.    |
| not                  | `not` is a modifier saying anything but x. For example, not UDP.                                         |
|                      |                                                                                                          |

Host filter
```sh
sudo tcpdump -i eth0 host 172.16.146.2
```

Dest filter
```sh
# Syntax: src/dst [host|net|port] [IP|Network Range|Port]
sudo tcpdump -i eth0 src host 172.16.146.2
```

Utilizing source with
```sh
# port as filter
sudo tcpdump -i eth0 tcp src port 80

# combination with net filter
sudo tcpdump -i eth0 dest net 172.16.146.0/24
```

Protocol filter
```sh
# name: tcp/udp/icmp
sudo tcpdump -i eth0 udp

# number
sudo tcpdump -i eth0 proto 17
```

Port filter
```sh
sudo tcpdump -i eth0 tcp port 443

# portrange
sudo tcpdump -i eth0 portrange 0-1024
```

Size filter (less/greater)
```sh
# Syntax: less/greater [size in bytes]
sudo tcpdump -i eth0 less 64
```

AND/OR filter
```sh
sudo tcpdump -i eth0 host 192.168.0.1 and port 23

sudo tcpdump -r sus.pcap icmp or host 172.16.146.1
```

NOT filter
```sh
sudo tcpdump -r sus.pcap not icmp
```

Apply filter pre-capture/post-capture?
- Pre-capture:
	- + Reduce amount of data in traffic
	- - Clear out traffic that may be needed
- Post-capture
	- + Might be important
	- - Lots of data

Show ascii values
```shell-session
sudo tcpdump -Ar telnet.pcap
```

Piping to grep
```shell-session
sudo tcpdump -Ar http.cap -l | grep 'mailto:*'
```

Looking for:
```sh
# TCP protocol
tcpdump -i eth0 'tcp[13] &2 != 0'

# SYN flag
sudo tcpdump -i eth0 'tcp[13] &2 != 0'
```

# Wireshark
## Analysis with Wireshark
Wireshark is multi-platform and capable of capturing live data off many different interface types (to include WiFi, USB, and Bluetooth) and saving the traffic to several different formats. What makes Wireshark truly powerful is the analysis capability it provides, giving a detailed insight into the traffic.

TShark is a purpose-built terminal tool based on Wireshark. TShark shares many of the same features that are included in Wireshark and even shares syntax and options. [Termshark](https://github.com/gcla/termshark) is the TUI alternative.
```sh
tshark -i eth0 -w /tmp/test.pcap
tshark -i eth0 -f "host 172.16.146.2" -w /tmp/test.pcap
```

|**Capture Filters**|**Result**|
|:-:|---|
|host x.x.x.x|Capture only traffic pertaining to a certain host|
|net x.x.x.x/24|Capture traffic to or from a specific network (using slash notation to specify the mask)|
|src/dst net x.x.x.x/24|Using src or dst net will only capture traffic sourcing from the specified network or destined to the target network|
|port #|will filter out all traffic except the port you specify|
|not port #|will capture everything except the port specified|
|port # and #|AND will concatenate your specified ports|
|portrange x-x|portrange will grab traffic from all ports within the range only|
|ip / ether / tcp|These filters will only grab traffic from specified protocol headers.|
|broadcast / multicast / unicast|Grabs a specific type of traffic. one to one, one to many, or one to all.|

|    **Display Filters**     | **Result**                                                                                    |
| :------------------------: | --------------------------------------------------------------------------------------------- |
|     ip.addr == x.x.x.x     | Capture only traffic pertaining to a certain host. This is an OR statement.                   |
|   ip.addr == x.x.x.x/24    | Capture traffic pertaining to a specific network. This is an OR statement.                    |
|   ip.src/dst == x.x.x.x    | Capture traffic to or from a specific host                                                    |
| dns / tcp / ftp / arp / ip | filter traffic by a specific protocol. There are many more options.                           |
|       tcp.port == x        | filter by a specific tcp port.                                                                |
|  tcp.port / udp.port != x  | will capture everything except the port specified                                             |
|       and / or / not       | AND will concatenate, OR will find either of two options, NOT will exclude your input option. |

## Wireshark Advanced Usage
### Plugins
- Statistics tab
- Analyze tab
- TCP Stream
- Extract files

#### FTP Filters
`ftp` - Will display anything about the FTP protocol.
`ftp.request.command` - Will show any commands sent across the ftp-control channel ( port 21 )
`ftp-data` - Will show any data transferred over the data channel ( port 20 )

## Packet Inception, Dissecting Network Traffic
...

Connect to RDP: 
```sh
xfreerdp /v:<target IP> /u:htb-student /p:HTB_@cademy_stdnt!
```

## Guided Lab

## Decrypting RDP Connections