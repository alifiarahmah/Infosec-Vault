# Network Enumeration with Nmap

> [!note]
> It is always recommended to store every single scan. This can later be used for comparison, documentation, and reporting. After all, different tools may produce different results.


# Host discovery

Scan network range
```sh
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

Scan from lists
```sh
sudo nmap -sn -oA tnet -iL hosts.txt | grep for | cut -d" " -f5
```

Scan multiple ips
```sh
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5
```

Single ip, store result in all format starting with name host
```sh
sudo nmap 10.129.2.18 -sn -oA host 
```

Packet trace
```sh
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

We see here that `Nmap` does indeed detect whether the host is alive or not through the `ARP request` and `ARP reply` alone. To disable ARP requests and scan our target with the desired `ICMP echo requests`, we can disable ARP pings by setting the "`--disable-arp-ping`" option
```sh
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping 
```
An `ICMP echo request` can help us determine if our target is alive and identify its system. ==Cek TTL di reply==

![](https://ostechnix.com/wp-content/uploads/2021/09/Operating-systems-TTL-Values.png.webp)

# Host and port scanning

| **State**          | **Description**                                                                                                                                                                                         |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `open`             | This indicates that the connection to the scanned port has been established. These connections can be **TCP connections**, **UDP datagrams** as well as **SCTP associations**.                          |
| `closed`           | When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an `RST` flag. This scanning method can also be used to determine if our target is alive or not. |
| `filtered`         | Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.                  |
| `unfiltered`       | This state of a port only occurs during the **TCP-ACK** scan and means that the port is accessible, but it cannot be determined whether it is open or closed.                                           |
| `open\|filtered`   | If we do not get a response for a specific port, `Nmap` will set it to that state. This indicates that a firewall or packet filter may protect the port.                                                |
| `closed\|filtered` | This state only occurs in the **IP ID idle** scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.                                           |
## Scanning Options

| **Nmap Option**      | **Description**                                                        |
| -------------------- | ---------------------------------------------------------------------- |
| `10.10.10.0/24`      | Target network range.                                                  |
| `-sn`                | Disables port scanning.                                                |
| `-Pn`                | Disables ICMP Echo Requests                                            |
| `-n`                 | Disables DNS Resolution.                                               |
| `-PE`                | Performs the ping scan by using ICMP Echo Requests against the target. |
| `--packet-trace`     | Shows all packets sent and received.                                   |
| `--reason`           | Displays the reason for a specific result.                             |
| `--disable-arp-ping` | Disables ARP Ping Requests.                                            |
| `--top-ports=<num>`  | Scans the specified top ports that have been defined as most frequent. |
| `-p-`                | Scan all ports.                                                        |
| `-p22-110`           | Scan all ports between 22 and 110.                                     |
| `-p22,25`            | Scans only the specified ports 22 and 25.                              |
| `-F`                 | Scans top 100 ports.                                                   |
| `-sS`                | Performs an TCP SYN-Scan.                                              |
| `-sA`                | Performs an TCP ACK-Scan.                                              |
| `-sU`                | Performs an UDP Scan.                                                  |
| `-sV`                | Scans the discovered services for their versions.                      |
| `-sC`                | Perform a Script Scan with scripts that are categorized as "default".  |
| `--script <script>`  | Performs a Script Scan by using the specified scripts.                 |
| `-O`                 | Performs an OS Detection Scan to determine the OS of the target.       |
| `-A`                 | Performs OS Detection, Service Detection, and traceroute scans.        |
| `-D RND:5`           | Sets the number of random Decoys that will be used to scan the target. |
| `-e`                 | Specifies the network interface that is used for the scan.             |
| `-S 10.10.10.200`    | Specifies the source IP address for the scan.                          |
| `-g`                 | Specifies the source port for the scan.                                |
| `--dns-server <ns>`  | DNS resolution is performed by using a specified name server.          |

## Output Options

| **Nmap Option** | **Description**                                                                   |
| --------------- | --------------------------------------------------------------------------------- |
| `-oA filename`  | Stores the results in all available formats starting with the name of "filename". |
| `-oN filename`  | Stores the results in normal format with the name "filename".                     |
| `-oG filename`  | Stores the results in "grepable" format with the name of "filename".              |
| `-oX filename`  | Stores the results in XML format with the name of "filename".                     |

## Performance Options

|**Nmap Option**|**Description**|
|---|---|
|`--max-retries <num>`|Sets the number of retries for scans of specific ports.|
|`--stats-every=5s`|Displays scan's status every 5 seconds.|
|`-v/-vv`|Displays verbose output during the scan.|
|`--initial-rtt-timeout 50ms`|Sets the specified time value as initial RTT timeout.|
|`--max-rtt-timeout 100ms`|Sets the specified time value as maximum RTT timeout.|
|`--min-rate 300`|Sets the number of packets that will be sent simultaneously.|
|`-T <0-5>`|Specifies the specific timing template.|

# Service enumeration



# Nmap scripting engine



# Bypass security (firewall, idps)

```sh
sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
```

```sh
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

| **Scanning Options** | **Description**                       |
| -------------------- | ------------------------------------- |
| `10.129.2.28`        | Scans the specified target.           |
| `-p 21,22,25`        | Scans only the specified ports.       |
| `-sS`                | Performs SYN scan on specified ports. |
| `-sA`                | Performs ACK scan on specified ports. |
| `-Pn`                | Disables ICMP Echo requests.          |
| `-n`                 | Disables DNS resolution.              |
| `--disable-arp-ping` | Disables ARP ping.                    |
| `--packet-trace`     | Shows all packets sent and received.  |

Scan using decoys

```sh
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

| **Scanning Options** | **Description**                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------ |
| `10.129.2.28`        | Scans the specified target.                                                                |
| `-p 80`              | Scans only the specified ports.                                                            |
| `-sS`                | Performs SYN scan on specified ports.                                                      |
| `-Pn`                | Disables ICMP Echo requests.                                                               |
| `-n`                 | Disables DNS resolution.                                                                   |
| `--disable-arp-ping` | Disables ARP ping.                                                                         |
| `--packet-trace`     | Shows all packets sent and received.                                                       |
| `-D RND:5`           | Generates five random IP addresses that indicates the source IP the connection comes from. |
### Connect to filtered port

```sh
ncat -nv --source-port 53 10.129.2.28 50000
```
