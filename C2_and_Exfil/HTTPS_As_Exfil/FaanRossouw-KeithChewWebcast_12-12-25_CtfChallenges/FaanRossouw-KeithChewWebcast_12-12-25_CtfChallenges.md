<a name="_hxuop7blruj4"></a>**Level 1: The Initial Triage**

**Context:** You’ve received an alert about a potential long-duration connection leaving the network. You have the PCAP.

**Question 1:** Filtering out the background noise (like the Ubuntu connectivity checks), what is the primary domain domain name the internal host is beaconing to?

- **Answer:** private.canadianshield.cira.ca

**Question 2:** Based on the User-Agent string found in the HTTP streams, what specific browser/OS is the C2 client pretending to be?

- **Answer:** Firefox 110.0 on Ubuntu Linux (Mozilla/5.0 (X11; Ubuntu; Linux x86\_64; rv:109.0) Gecko/20100101 Firefox/110.0).

![ref1]

<a name="_nsg8cw1zg8q0"></a>**Level 2: Traffic Analysis**

**Context:** We know where it's going. Now let's look at *how* it's communicating.

**Question 3:** Ligolo-ng establishes a tunnel. Looking at the SSL/TLS certificate exchange (specifically the "Server Hello"), who is the issuer of the certificate used for this C2 channel?

- **Answer:** Let's Encrypt (R3).

**Question 4:** Ligolo-ng traffic (or the C2 traffic inside it) often has a heartbeat or "jitter." In this capture, we see a distinct ASCII pattern in the payload bytes that increments. What is the 3-letter sequence seen repeating in the data payload?

- **Answer:** It cycles through TRK, TRL, TRM, TRN... (The letters increment: K, L, M, N, etc.).

![ref1]

<a name="_qdrkyt5enh4e"></a>**Level 3: The Deep Dive**

**Context:** You need to write a Snort/Suricata signature or a detection rule for this specific campaign.

**Question 5:** We see a lot of TCP Keep-Alive packets and 1-byte segments (or empty segments) in the stream. Why is this characteristic of a tunneling tool like Ligolo-ng, specifically?

- **Answer:** Ligolo-ng creates a persistent TCP (or UDP) connection to handle the tunnel interface. Unlike a standard beacon that sleeps and disconnects, a tunnel tool tries to keep the pipe open. The PCAP shows long-duration sessions with periodic "keep-alives" to prevent the firewall from killing the state tables.

**Question 6:** If we assume the traffic inside the tunnel is encrypted, but we can see the size of the packets, how can we tell if the attacker is actively running commands vs. the tunnel just sitting idle?

- **Answer:** Look at the Packet Length (frame size) variance.
- **Analysis:** Throughout the file, we see blocks of small, uniform packets (Heartbeats/Idle) and then sudden bursts of larger packets (likely the "Command & Convo" mentioned in your Episode title). Even without decrypting the tunnel, the volume and size of packets betray the *timing* of the attacker's activity.


[ref1]: Aspose.Words.105b93f6-7423-4043-9eff-6763bbee6e44.001.png
