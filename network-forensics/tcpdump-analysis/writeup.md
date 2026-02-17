# Objective

- Detect and analyze a possible SYN Flood attack through tcpdump, identifying anomalous patterns in the TCP handshake.

---

# Environment

- Attacker: 172.17.0.1 (Local VM / Kali Linux)  
- Target machine: 172.17.0.2 (docker0)  
- Local network  
- Tools: tcpdump, Snort  

---

# Simulated Scenario

- An attacker scanning vulnerable ports trying to exploit open ports on the target.

---

# Attack Execution (Lab Activity)

- nmap -p 21,23,80 172.17.0.2  
- Scan on multiple ports of target 172.17.0.2 (docker0) trying to identify open ports.

### Results:

PORT   STATE   SERVICE  
21/tcp closed  ftp  
23/tcp closed  telnet  
80/tcp open    http  

---

# Traffic Capture Procedure

- As a SOC analyst, I used two tools to detect and capture packets from the host: tcpdump and Snort.  

- I received three alerts from Snort:

02/15-12:47:46.449547 [**] [1:1000004:0] "NMAP Scan to HTTP(80) Port Detected" [**] [Priority: 0] {TCP} 172.17.0.1:42877 -> 172.17.0.2:80  
The host 172.17.0.1 on port 42877 scanned port 80 on host 172.17.0.2  

02/15-12:47:46.449753 [**] [1:1000005:0] "NMAP Scan to Telnet(23) Port Detected" [**] [Priority: 0] {TCP} 172.17.0.1:42877 -> 172.17.0.2:23  
The host 172.17.0.1 on port 42877 scanned port 23 on host 172.17.0.2  

02/15-12:47:46.449770 [**] [1:1000003:0] "NMAP Scan to FTP(21) Port Detected" [**] [Priority: 0] {TCP} 172.17.0.1:42877 -> 172.17.0.2:21  
The host 172.17.0.1 on port 42877 scanned port 21 on host 172.17.0.2  

- Snort generated alerts. As a SOC analyst, I performed a deeper analysis.  
I configured tcpdump to capture TCP traffic destined for host 172.17.0.2 containing the SYN flag (which represents the beginning of a TCP three-way handshake):

sudo tcpdump -i docker0 -v tcp and host 172.17.0.2 and 'tcp[tcpflags] & tcp-syn != 0'

- After Snort generated alerts, I reviewed the historical tcpdump capture located at tcpdump/capture.pcap.  
To be more specific, I filtered the packets to identify SYN packets without ACK:

sudo tcpdump -r tcpdump/capture.pcap -v 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0' and host 172.17.0.2

172.17.0.1.33003 > 172.17.0.2.ssh: Flags [S], cksum 0x9561 (correct), seq 0:22, win 8192, length 22  
18:02:26.698003 IP (tos 0x0, ttl 64, id 1, offset 0, flags [none], proto TCP (6), length 62)  
172.17.0.1.33004 > 172.17.0.2.ftp: Flags [S], cksum 0x54a4 (correct), seq 0:22, win 8192, length 22: FTP, length: 22  

This result means that the attacker sent TCP SYN packets to the target on port 22 and port 21, and the target machine responded with TCP RST packets, rejecting the connection attempt.

- Snort alerted on three port scans. Using this filter, we observed two. This means the third scan on the HTTP port was accepted by the target, which responded with TCP SYN-ACK, indicating that the service was open.

---

# Defensive View

- Avoid leaving vulnerable ports open.  
- Always use IDS or IPS solutions.  
- Avoid weak services like HTTP; enforce HTTPS instead (HSTS).  
- Implement firewall rules and ACLs.
- Disable unused services
- implementing rate limiting
- configure port scan detection threshold

---

# ISO 27001 and ISO 27002

- Ensure compliance with Domain 12 of ISO 27001. Logging and monitoring are fundamental for network forensics.  
- Domain 13 of ISO 27001 emphasizes secure communication. Organizations should implement defensive tools such as firewalls, encrypted communication channels, and proper network segmentation.
