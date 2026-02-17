# Credentials Breach Investigation — Wireshark Forensic Analysis

**Author:** matheusg-sec
**Date:** February 2025
**Tool(s):** Wireshark
**Category:** Network Forensics

---

## 1. Context

> Your organization has been alerted that a sensitive file, `credentials.txt`, hosted on an internal nginx web server, has been found leaked on the dark web. The file contains login credentials and API tokens used by internal tools. A PCAPNG capture from the company's monitoring system was handed over to conduct a forensic investigation.

---

## 2. Objective

> - Identify how the credentials were accessed
> - Determine whether they were exfiltrated
> - Provide undeniable proof that the credentials were indeed leaked

---

## 3. Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Wireshark | 4.x | Packet analysis and display filters |

---

## 4. Investigation Process

### 4.1 — First Observation (Overview)

> I opened the `credentials-breach.pcapng` file in Wireshark and identified three distinct traffic types:
> - HTTP traffic between internal hosts
> - DNS queries from an internal IP to Google DNS to resolve an external domain
> - An internal host communicating with an external host via TCP

> The combination of internal HTTP access followed by an external DNS query and outbound TCP connection immediately caught my attention as a potential exfiltration pattern.

---

### 4.2 — Initial Hypothesis

> Based on the traffic, an internal host made an HTTP request to the nginx container. The container responded with HTTP 200 OK and delivered `credentials.txt` in the payload. Right after that, the same internal host initiated communication with an external host, suggesting the credentials were exfiltrated.

---

### 4.3 — Deep Investigation

**Investigating - Part 1 — Credential Access**
```
The credentials were accessed by host 172.17.0.1 through an HTTP request to 172.17.0.2 (nginx container).
The container responded with HTTP 200 OK and delivered credentials.txt within the payload.
```

**Investigating - Part 2 — External DNS Resolution**
```
After receiving credentials.txt, host 172.17.0.2 sent a DNS query to 8.8.8.8 (Google DNS)
to resolve an external domain: malicious.attackdomain.com
8.8.8.8 responded with the resolved IP address: 201.20.20.30
```

**Investigating - Part 3 — Outbound TCP Connection**
```
Right after the DNS resolution, host 172.17.0.1 initiated a TCP SYN connection to 201.20.20.30.
The payload sent within this connection contained the contents of credentials.txt in plaintext.
Host 201.20.20.30 responded with TCP SYN-ACK, confirming the connection was established.
```

---

### 4.4 — Evidence Found

```
Frame 8 | 0.003289s | 172.17.0.1 → 201.20.20.30 | TCP | 33333 → 4444 [SYN] Seq=0 Win=8192 Len=0

Payload:
  Exfiltrating credentials.txt:
  username: testadmin
  password: P@ssw0rd123!
  token: 9f35-ef91-ab12
```

---

## 5. Conclusion

**Event Classification:** Incident — Confirmed Credentials Exfiltration

**CIA Impact:**
- **Confidentiality:** Affected — Sensitive credentials and API tokens were transmitted in plaintext to an external IP
- **Integrity:** Not directly observed — No evidence of data modification in the capture, however if credentials were changed by the attacker, integrity would be compromised
- **Availability:** Not directly observed — If the attacker uses the stolen credentials to lock out legitimate users, availability would be affected

**Kill Chain Phases:**
- [X] Reconnaissance — Internal host accessed the sensitive file
- [ ] Weaponization
- [ ] Delivery
- [ ] Exploitation
- [ ] Installation
- [ ] Command & Control
- [X] Actions on Objectives — Credentials exfiltrated to external C2 server

> **Note:** Based on the available PCAP, only Reconnaissance and Actions on Objectives phases were directly observed. Earlier phases may have occurred prior to the capture window and would require further investigation.

---

## 6. Mitigation & Recommendations

**Detection:**
- IDS rule: Alert on outbound TCP connections to non-standard ports (e.g. 4444, 1337, 9999)
- Firewall log: Flag and review any outbound connections to unknown external IPs
- SIEM: Correlate DNS queries to external domains with subsequent outbound TCP connections from the same host

**Prevention:**
- Enforce HTTPS on all internal services — HTTP exposes data in plaintext
- Restrict access to sensitive files — only authorized hosts should be able to reach them
- Implement egress filtering — block outbound connections to ports and destinations not explicitly authorized
- Enable DNS filtering to block resolution of known malicious domains

---

## 7. Lessons Learned

- Sensitive files should never be served over unencrypted HTTP
- Monitoring outbound DNS queries is a valuable early indicator of exfiltration attempts
- The sequence HTTP access → external DNS query → outbound TCP is a classic exfiltration pattern worth adding to detection rules

---

## 8. References

- Logix Academy — Room: Credentials Breach Investigation

---

*Write-up by matheusg-sec | February 2025*
