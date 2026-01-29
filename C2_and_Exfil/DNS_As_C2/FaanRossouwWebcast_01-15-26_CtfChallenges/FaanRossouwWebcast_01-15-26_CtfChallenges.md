DNS TXT C2 Investigation Lab

Case Study: Joker Screenmate — TXT Record Abuse in DNS C2

**CHALLENGES**

**🔴 Challenge 1 — Identify Suspicious DNS TXT Record Abuse**

Identify which domain is abusing DNS TXT records and the internal host generating the suspicious traffic.

**Solution 1 — Commands & Reasoning**

Step 1 — Identify TXT-heavy domains\
` `cat dns.log | zeek-cut query qtype\_name | grep TXT | sort | uniq -c | sort -nr\
\
Expected result:

4696 verify.timeserversync.com       TXT\
\
Step 2 — Identify the internal host\
` `cat dns.log | zeek-cut id.orig\_h query qtype\_name | grep verify.timeserversync.com | grep TXT | sort | uniq -c\
\
Expected result:

4696 192.168.2.88    verify.timeserversync.com       TXT

**🔴 Challenge 2 — Inspect DNS TXT Responses**

Your task:\
` `Inspect the TXT responses returned by the suspicious domain and determine whether the responses resemble encoded or chunked data.

**Solution 2 — Commands & Reasoning**

Inspect TXT answers\
` `cat dns.log | zeek-cut query qtype\_name answers | grep verify.timeserversync.com | grep TXT\
\
Observed characteristics:\
` `- Long TXT strings\
` `- Similar lengths\
` `- Repeated structure

**🔴 Challenge 3 — Discover Additional HTTPS Communication**

Your task:\
` `Determine whether the compromised host established **non-DNS connections** to the same external IP used as the authoritative DNS server, identify which protocol(s) were used. Assess whether this indicates additional command-and-control activity.

**Solution 3 — Commands & Reasoning**

Search for connections to the authoritative nameserver IP, excluding DNS traffic\
` `cat conn.log | zeek-cut uid id.resp\_h service duration orig\_bytes resp\_bytes history \

| grep 48.217.188.16 | grep -v dns\
\
Expected observations: connections from 192.168.2.88 using ssl or https with sustained sessions and non-trivial byte counts.\
` `Conclusion: the same external IP is used for both DNS-based activity and HTTPS communication, indicating multi-protocol C2 infrastructure rather than DNS-only behavior. 

**🔴 Challenge 4 — Identify DNS Query Destinations**

Determine where DNS queries for the suspicious domain are being sent, whether the infected host is querying a recursive resolver or contacting an authoritative nameserver directly, and what this behavior indicates about the malware DNS usage.

**Solution 4 — Commands & Reasoning**

Identify DNS responder IPs for the suspicious domain\
` `cat dns.log | zeek-cut id.resp\_h query qtype\_name \ | grep timeserversync.com | sort |      uniq -c\
\
Expected observations: DNS queries are sent directly to a single external IP acting as an authoritative nameserver rather than an internal recursive resolver. Conclusion: the infected host bypasses normal DNS resolution and communicates directly with attacker-controlled infrastructure, enabling reliable tracking and faster command-and-control.
