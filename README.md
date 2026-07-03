# Ransomware Incident Investigation — ProcDOT & Network Forensics

## Overview

The firewall flagged a machine in the Sales department — a system that stores customer
data — for contacting known malicious domains. Analysts noticed the outbound traffic
contained suspicious base64-encoded strings, which pulled the Incident Response team into
the case. By the time responders reached the machine, the ransomware had already announced
itself: a changed desktop wallpaper and a ransom note sitting on disk.

The job was to go beyond "we've been hit" and build an actual forensic trail — identify the
ransomware family, trace how it moved on the host, and pull the network indicators of
compromise using process monitoring data and a packet capture.

**Type:** Malware Analysis / Incident Response
**Platform:** TryHackMe
**Tools:** ProcDOT, Wireshark, AlienVault OTX

---

## 1. Reconstructing Process Activity with ProcDOT

Loaded the Process Monitor log and the network capture into ProcDOT to visually reconstruct
what happened on the host. Started by selecting the relevant process from the full process
list to anchor the graph around the point of compromise.

## 2. Visualizing the Attack Timeline

ProcDOT renders process, file, and network activity as a live timeline graph, letting you
scrub through the incident second by second instead of reading raw log lines. This surfaced
the sequence of process spawns and network connections around the infection point.

## 3. Mapping Network Connections

Rendered the full connection graph, showing outbound traffic to multiple external IPs
alongside registry activity tied to certificate issuers — a strong signal of the malware
attempting to blend in with legitimate Windows processes while phoning home.

## 4. Identifying the Malicious Process

Drilled into a suspicious `explorer.exe` process running from an unusual path —
`C:\Users\sales\AppData\Local\Temp\explorer.exe` — a classic masquerade technique, since
the legitimate `explorer.exe` never runs from a Temp directory. Process details confirmed
its full path, command line, parent PID, and thread count, marking it as the core malicious
process behind the incident.

## 5. Analyzing the Network Capture

Opened the corresponding `traffic.pcap` in Wireshark to correlate the process-level findings
with actual network traffic, filtering for TLS and TCP sessions tied to the infected host.

## 6. Following the TCP Stream

Followed the TCP stream for one of the flagged connections and found an HTTP redirect
chain pointing to `mojobiden[.]com`, with a JavaScript redirect masquerading as an OpenDNS
phishing block page (`phish.opendns.com`) — a technique used to disguise C2 traffic as a
benign security block, and a strong lead on the malware's callback infrastructure.

## 7. Threat Intelligence Correlation

Took the identified file hash and domains to AlienVault OTX for correlation. The lookup
confirmed the sample was tied to **BlackMatter ransomware**, flagged across 50 threat
intelligence pulses, with matching YARA rules for `dead_host`, `ransomnote_file_moves`,
`ransomware_appends_extension`, and `network_scrp_dumped_buffer` — all consistent with
BlackMatter's known behavior of dropping ransom notes, appending custom file extensions,
and scanning the network for lateral movement targets.

---

## Findings

- The infected host was running a disguised malicious process (`explorer.exe`) executing
  from a Temp directory rather than its legitimate system location.
- Network traffic showed outbound connections to multiple malicious IPs and a C2 domain
  (`mojobiden[.]com`) disguised behind a fake OpenDNS phishing redirect.
- Threat intelligence correlation via AlienVault OTX confirmed the ransomware family as
  **BlackMatter**, based on file hash, YARA detections, and known IOC overlap.
- Process and network evidence combined gave a complete picture: initial execution,
  masquerading, C2 communication, and eventual file encryption — consistent with a full
  ransomware deployment chain rather than an isolated alert.

## Key Takeaway

Endpoint telemetry and network traffic tell two halves of the same story — ProcDOT showed
what happened on the host, and the packet capture showed where the traffic was actually
going. Neither alone would have been enough to confirm BlackMatter as the ransomware
family; it took correlating process behavior, network IOCs, and threat intelligence
together to build a defensible conclusion.

## Skills Demonstrated

`Malware Analysis` `Process Monitoring` `Network Traffic Analysis`
`Threat Intelligence Correlation` `Ransomware Identification` `Incident Response`

---

## Screenshots

![screenshot1](https://github.com/aBdUl-AhaD02/Ransomware-Incident-Investigation/blob/main/Images/Screenshot%2001.png)

![screenshot2](https://github.com/aBdUl-AhaD02/Ransomware-Incident-Investigation/blob/main/Images/Screenshot%2002.png)

![screenshot3](https://github.com/aBdUl-AhaD02/Ransomware-Incident-Investigation/blob/main/Images/Screenshot%2003.png)

![screenshot4](https://github.com/aBdUl-AhaD02/Ransomware-Incident-Investigation/blob/main/Images/Screenshot%2004.png)

![screenshot5](https://github.com/aBdUl-AhaD02/Ransomware-Incident-Investigation/blob/main/Images/Screenshot%2005.png)

![screenshot6](https://github.com/aBdUl-AhaD02/Ransomware-Incident-Investigation/blob/main/Images/Screenshot%2006.png)

![screenshot7](https://github.com/aBdUl-AhaD02/Ransomware-Incident-Investigation/blob/main/Images/Screenshot%2007.png)
