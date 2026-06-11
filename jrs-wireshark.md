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
