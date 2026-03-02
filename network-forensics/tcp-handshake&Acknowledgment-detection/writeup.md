
# Network-Analysis-01: TCP Handshake & Reconnaissance Detection

## 1. Executive Summary
During a routine network monitoring session, the IDS (Intrusion Detection System) triggered an alert regarding abnormal traffic patterns. Further analysis revealed a massive volume of **TCP-SYN** packets originating from multiple external sources. This behavior is consistent with a **TCP SYN Flood (DDoS)** attempt or a wide-scale **SYN Scan** (Reconnaissance phase), targeting internal services to identify open ports and exhaust system resources.

## 2. Technical Background
Understanding the **TCP 3-Way Handshake** is critical for identifying this anomaly. Under normal conditions:
1.  **SYN**: Host A sends a synchronization request to Host B.
2.  **SYN-ACK**: Host B acknowledges and sends its own sync request.
3.  **ACK**: Host A confirms, establishing the connection.

In this case, the attacker utilizes a **Half-Open (SYN) Scan**. The attacker sends the initial SYN and receives the SYN-ACK, but instead of completing the handshake with an ACK, they send a **RST (Reset)** packet. This allows them to confirm the port is open without creating a full connection, making it harder for simple logging systems to track.

## 3. Toolset
* **Wireshark:** Primary tool for deep packet inspection and traffic filtering.
* **Nmap:** Used to simulate the scanning activity for baseline comparison.
* **IDS Alerts:** Triggering mechanism for the investigation.

## 4. Methodology & Execution
Following the IDS alert, the investigation followed these steps:
1.  **Protocol Filtering:** Applied a global filter for the `TCP` protocol to visualize the traffic spike.
2.  **Handshake Analysis:** Identified thousands of connection attempts lacking the final `ACK` from the source IP.
3.  **Filter Implementation:** To isolate the suspicious activity, I utilized the following display filters:
    * `tcp.flags.syn == 1 && tcp.flags.ack == 0`: To see incoming connection requests.
    * `tcp.flags.reset == 1`: To detect the immediate termination after a SYN-ACK response.
4.  **Pattern Recognition:** The high frequency of `SYN` followed by `RST` from the same source confirmed a stealthy sweep (SYN Scan).

## 5. Evidence & Logs
The following log represents the "smoking gun" — a successful identification of a port 22 (SSH) scan:

| TIME | SOURCE | DESTINATION | PROTOCOL | INFO |
| :--- | :--- | :--- | :--- | :--- |
| 10:05:01 | 10.0.0.5 | 192.168.1.10 | TCP | 41656 → 22 [SYN] Seq=0 |
| 10:05:01 | 192.168.1.10 | 10.0.0.5 | TCP | 22 → 41656 [SYN, ACK] Seq=0 Ack=1 |
| 10:05:01 | 10.0.0.5 | 192.168.1.10 | TCP | 41656 → 22 [RST] Seq=1 |

*Observation: The RST packet sent by 10.0.0.5 immediately after the server's SYN-ACK is the technical signature of a SYN Scan.*

## 6. Remediation Strategy
To protect the infrastructure against these threats, the following controls should be implemented:
* **Rate Limiting (Technical):** Implement rate limiting by IP address on the perimeter firewall to mitigate SYN Floods and slow down automated scans.
* **Cloud-Based Mitigation:** Leverage cloud DDoS protection services (e.g., Cloudflare, AWS Shield) to scrub malicious traffic before it reaches the internal network.
* **Load Balancing:** Deploy a load balancer to distribute traffic and prevent a single server from being overwhelmed by SYN requests.
* **Hardening (Administrative):** Enforce a strict "Least Privilege" port policy, closing all non-essential services to reduce the attack surface.

---
**Documented by:** Matheus Gomes
**Role:** Cybersecurity Analyst (Aspirant)
**Date:** March 02, 2026
