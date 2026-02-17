# Credentials Breach Investigation — Wireshark Forensic Analysis

**Author:** matheusg-sec
**Date:** February 2025
**Tool(s):** Wireshark 
**Category:** Network Forensics 

---

## 1. Context

> Your organization has been alerted that a sensitive file, credentials.txt, hosted on an
internal nginx web server, has been found leaked on the dark web.
The file contains login credentials and API tokens used by internal tools.

---

## 2. Objective

> Identify how the credentials were accessed
> Determine wheather they were exfil
> Provide undeniable proof that the credentials were indeed leaked


---

## 3. Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Wireshark | 4.x | Packet analysis and display filters |

---

## 4. Investigation Process

### 4.1 — First Observation (Overview)

> I opended the credentials-breach.pcapng within wireshark

```bash
I analysed HTTP traffic between internal host, DNS Querys from an internal IP to google DNS
to resolve an external domain and the internal host comunicating with a external host 
```

> What did you observe? Describe what you saw and what caught your attention.

```bash
An internal IP communicating with a external host. 
```

---

### 4.2 — Initial Hypothesis

> Based on the traffic, the interal host made a HTTP request to ngix container, after that the container reponded
with status 200 OK, and with the HTML/Text, in sequence this same internal host started a comunication with a external host 

---

### 4.3 — Deep Investigation

> I found a suspicius traffic coming from this credentials-breach.pcapng, let's conduce a deep analysis

**Investigating - Part 1 
```
The credentials were accessed by the host 172.17.0.1 though an http request to 172.17.0.1 ( nginx container ) 
the host 172.17.0.1 responded with a http response status 200 OK, and the host also sent the credentials.txt within the payload 

```

**Investigating - Part 2
```
After the response from 172.17.0.1 with credentials.txt, the host 172.17.0.2 sent a DNS query to 8.8.8.8 ( Google DNS  ) to resolve an external domain, named malicious.attackdomain.com
host 8.8.8.8 responded to 172.17.0.2 with the IP address from the external domain ( 201.20.20.30 ) 
```

**Investigating - Part 3
```
Right after that the host 172.17.0.1 initiated a TCP syn connection with the host 201.20.20.30 and the payload that has sent has the credentials.txt file within
The host 201.20.20.30 responded with a TCP SYN, ACK confirming that the host has received the packet from 172.17.0.1 

```
 
### 4.4 — Evidence Found

- 8	0.003289	172.17.0.1	201.20.20.30	TCP	54	33333 → 4444 [SYN] Seq=0 Win=8192 Len=0
- Payload: 
  xfiltrating credentials.txt:
  username: testadmin
  password: P@ssw0rd123!
  token: 9f35-ef91-ab12


---

## 5. Conclusion

> What did you conclude from the full investigation?

**Event Classification:** [Normal Traffic / Reconnaissance / Attack / Incident]
-

**CIA Impact:**
- **Confidentiality:** [Affected] — A external IP received a sensite file from a entity 
- **Integrity:** [It wasn't clear] — In thas case it was't clear, if he change the crendentials it would affect the integrity 
- **Availability:** [It wasn't clear] —  In thas case it was't clear, if he change the crendentials this service would be unavalability for legitime users

**Kill Chain Phase:**
- [X] Reconnaissance
- [ ] Weaponization
- [ ] Delivery
- [ ] Exploitation
- [ ] Installation
- [ ] Command & Control
- [X] Actions on Objectives

---

## 6. Mitigation & Recommendations

> How could this type of activity be detected or prevented?

**Detection:**
- IDS rule: Alert on outbound TCP connections to non-standart ports 
- Firewall log
- SIEM: correlate events from a external IP

**Prevention:**
- Implement HSTS politicy
- deactivating vulnerable ports 
- Ensure that just authorized host has access to sentive files 

---

## 7. Lessons Learned

- It's always good to ensure that just authorized users acess sentive datas,
we should always use encriptation on sensitive files, it's a good pratice keep monitoring the traffic coming from the hosts
of your network

---

## 8. References

-  Logix academy 
-  Room name: Credentials Breach Investigation 

---

*Write-up by matheusg-sec | 17/02/2026 *
