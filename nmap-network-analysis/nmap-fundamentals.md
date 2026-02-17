# Nmap Fundamentals — Project 0

## 1. Overview
Daily scan performed against my default gateway.

---

## 2. Environment
- Local network environment

---

## 3. Scan Scenarios

### Targeted Port Scan (-p)

This scan was performed against my default gateway IP address. The objective was to identify exposed services and potential attack surface within my network.

During the scan, I observed a suspicious scenario where port **23/tcp** appeared as **FILTERED**. Port 23 is commonly targeted due to the use of the Telnet protocol, which transmits data in clear text when misconfigured or exposed.

The **FILTERED** status indicates that the traffic was blocked or silently discarded by a security control such as a firewall or another defensive mechanism. This means that the scan was unable to determine whether the service was running, suggesting the presence of network-level filtering.

---

### Source Port Manipulation (-g)

In this scenario, I used the same target IP address as the previous scan, but this time the scan originated from source port **53**, which is commonly associated with DNS traffic.

This technique is sometimes used because security devices such as firewalls or IDS/IPS may allow traffic based on permissive port rules rather than full contextual inspection. However, even when using a common port like 53, security controls should still inspect the traffic.

I observed that port **23/tcp** continued to appear as **FILTERED**, indicating that the security controls in place were not bypassed. This suggests that the network is using a **stateful firewall**, capable of inspecting traffic beyond simple port-based rules and applying deeper inspection to each packet.

---

## 4. Analysis and Interpretation

The use of the `-g 53` option was ineffective in this case because the network firewall was operating in a **stateful mode**.

The **FILTERED** status reveals that packets were being blocked or silently discarded by the firewall, preventing the scan from determining whether the service was active.

---

## 5. Defensive Perspective

- A SOC analyst would observe that the firewall is operating correctly in **stateful mode**, indicating that security controls are functioning as expected.  
- An important point to investigate would be the **origin of the traffic targeting port 23**, determining whether it originated from an internal host or an external source.  
- The team should perform additional network analysis to **identify active hosts** and assess whether any internal system is attempting to access or expose insecure services.

---

## 6. What I Learned

- In this scan, I observed a simple evasion technique using a **common port**.  
- I also learned how important it is for security mechanisms to operate in **stateful mode**, as this allows the network to block basic evasion attempts like source port manipulation.
