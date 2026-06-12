|  ~	| Filter                                                       	| Function                                                                     	|
|-----------	|----------------------------------------------------------	|---------------------------------------------------------------------------------------------	|
| Wireshark 	| ip                                                       	| Shows the # of packets.                                                                     	|
| Wireshark 	| ip.ttl < 10                                              	| Shows the # of packets with a TTL value less than 10.                                       	|
| Wireshark 	| tcp.port==4444                                           	| Shows the # of packets which use TCP port 4444.                                             	|
| Wireshark 	| http.request.method==GET && tcp.port==80                 	| Shows the # of "HTTP GET" requests sent to port 80.                                         	|
| Wireshark 	| dns.a                                                    	| Shows the # of type A DNS queries.                                                          	|
| Wireshark 	| http.server contains "Apache"                            	| Find all Apache servers. The "contain" filter searches for a value inside of packets.       	|
| Wireshark 	| http.host matches "\.(php\|html)"                        	| Find all .php and .html pages.                                                              	|
| Wireshark 	| tcp.port in {80 443 8080}                                	| Find all packets that use ports 80, 443 or 8080.                                            	|
| Wireshark 	| http.server contains "IIS" && http.server contains "7.5" 	| Find all Microsoft IIS servers and determine the number of packets that have "version 7.5." 	|
| Wireshark 	| tcp.port==3333 or tcp.port==4444 or tcp.port==9999       	| Combine tcp.port lookups to check multiple ports.                                           	|
| Wireshark 	| string(ip.ttl) matches "[02468]$"                        	| Find the number of packets with even TTL values.                                            	|

## Additional Information

### General purpose of Wireshark (in a DCO context):
Isolate malicious artifacts.
Identify lateral movement.
Identify command-and-control traffic.

### Cut Down Background Noise

Highly recommended to hide normal, expected traffic in order to focus on active communication protocols (like TCP, HTTP, etc.) and anomalies. The following display filter will significantly cut down on excess data that is displayed.

`!(arp || dns || ntp || icmp)`

### Display Filters for DCO

#### Recon - Detect potential network mapping or brute-force attacks.

* Identify TCP connection requests without an acknowledgment:
`tcp.flags.syn == 1 && tcp.flags.ack == 0`

* Identify high volumes of immediate connection terminations:
`tcp.flags.reset == 1`

#### Command & Control (C2) & Exfiltration - is a host communicating or staging data?

* Isolate traffic from an internal subnet to an outside target:
`ip.src == 10.0.0.0/8 && !ip.dst == 10.0.0.0/8`

* Find specific TLS handshakes or SNIs (Server Name Indication) -- beaconing patterns:
`tls.handshake.type == 1`

* Look for abnormally long DNS queries (DNS Tunneling / exfiltration):
`dns.qry.name.len > 50 and !mdns`

#### Lateral Movement Exploitation - web attacks, unauthorized access, plain-text credentials.

* SQL Injection (SQLi) Patterns:
`http.request.uri contains "SELECT" || http.request.uri contains "UNION"`

* Cleartext Password Searching:
`frame contains "password" || frame contains "pwd" || frame contains "login"`

* Identify malicious web shells, file uploads, or credential submissions:
`http.request.method == "POST"`

## Workflow

* **Protocol Hierarchy:** Start with **Statistics >> Protocol Hierarchy**. Look for weird or unusual percentages. For example, if 90% of your traffic is data inside unexpected protocols like ICMP or DNS, you potentially have data exfiltration or a tunnel established.

* **Conversations:** Next, use **Statistics >> Conversations**. Switch to the **IPv4** or **TCP** tabs and sort by **Bytes**. The highest byte counts between an internal asset and an unknown public IP indicate your primary exfiltration or C2 targets.

* **Get Context:** 
	1. Right click a suspicious packet.
	2. Go **Follow > TCP Stream** or HTTP/TLS Stream.
		* Red text = traffic sent from the source.
		* Blue text = traffic sent from the destination.

* **Extract Malicious Artifacts:** export any artifacts (using **File >> Export Object**), ensuring that it is done inside of an isolated sandbox or system.

## What makes a packet suspcious?

Generally, this is when the metadata, structure, or content deviates from the expected.

Suspicious packets can be categorized in these groups:

1. Anomalous Flags, Protocol Violations
	* Invalid Flag Combinations, like the "Christmas Tree" packet [ FIN, URG, and PSH ], used by Nmap to sneak past firewalls.
	* Mismatched Handshakes: Receiving a SYN-ACK packet from an external IP when your internal host never sent an initial SYN packet. Might indicate DDOS or IP spoofing.
	* Weird Headers -- missing fields, invalid checksums.

2. Suspicious Context or Origin
	* Non-Standard Port Usage.
	* Unusual Internal Directions (possible lateral movement).
	* Known Malicious Infrastructure: Packets originating from or destined for IP addresses flagged on Threat Intelligence feeds


3. Structural Anomalies
	* Abnormal Payload-to-Header Ratio. For example, DNS query packets should be small. A large payload would be suspicious and would point toward DNS tunneling. 
	* High-Frequency Repeating Patterns AKA Beaconing.

4. Malicious Application Payload (Content)
	* Exploit Strings & Signatures: Text inside an HTTP request containing SQL syntax (e.g., `'1'='1`), directory traversal attempts (`../../etc/passwd`), or cross-site scripting tags (`<script>`).
	* Reverse Shell Indicators: Interactive shell artifacts where they don't belong, such as seeing `root#` or `whoami` traveling over a random, unencrypted TCP port.

*the end*
