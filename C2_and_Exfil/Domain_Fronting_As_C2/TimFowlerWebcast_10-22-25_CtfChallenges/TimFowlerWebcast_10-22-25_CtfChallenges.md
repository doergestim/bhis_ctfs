<a name="_m23i9wjisfr3"></a>**Red Team**

<a name="_9limvmfthlwt"></a>**1. Reconnaissance & Footprinting**

Your first goal is to build a complete profile of the target satellite and its ground network.

- **RF Analysis:** Can you passively identify the satellite's downlink frequency, modulation, and data rate? What public tools ( SDR, public tracking sites) can you use to predict its pass-over times and "listen" for its signal?
- **Ground Station Mapping:** Can you identify the physical locations of the target's ground stations? Are they in remote, physically insecure areas?
- **Network Enumeration:** What is the ground station's public IP address range? What ports or services ( FTP, SSH, web portals) are exposed to the internet? Can you find any leaked operator credentials on the dark web or in public code repositories?
- **Protocol Identification:** Can you capture enough raw signal to determine if they are using a standard protocol (like CCSDS) or a proprietary one? Is the telemetry encrypted or in the clear?

<a name="_wzv9bvnxwe2m"></a>**2. Vulnerability Analysis**

Now that you know *where* they are, find the weak points.

- **Encryption Weakness:** If the telemetry is encrypted, what algorithm is it? (You may infer this from signal properties or public mission documents). Can you mount an offline attack if you capture enough data? More importantly, is their **key management** weak? ( are keys hard-coded in ground station software you can find online?).
- **Ground Segment:** Is the ground station's control software (the "Mission Operations Center" or MOC) vulnerable? Is it an off-the-shelf product with known CVEs? Is their "data" server (where telemetry is stored) vulnerable to SQL injection or a simple default password?
- **Replay Vulnerability:** Capture a valid telemetry stream. Does the satellite's protocol use nonces or timestamps? If not, can you successfully execute a **replay attack**?
- **Authentication Flaw:** Can you reverse-engineer the "handshake" used for telemetry? Is it just a simple, static "pre-shared key" that you can forge? What's stopping you from spoofing a packet?

<a name="_sts5rvs9tdsp"></a>**3. Attack & Exploitation Scenarios**

Execute the attack.

- **Objective: Eavesdrop (Confidentiality Attack):**
  - **Challenge:** Intercept, demodulate, and (if necessary) decrypt the live telemetry feed for one full orbit.
  - **Goal:** Deliver a "live report" of the satellite's battery voltage, orientation, and current task *before* the legitimate operators do.
- **Objective: Jam (Availability Attack):**
  - **Challenge:** During a critical operational window ( when the satellite is downloading valuable science data), can you successfully jam the downlink signal?
  - **Goal:** Force the ground station to log a "Loss of Signal" (LOS) for at least 60 seconds, causing the data download to fail.
- **Objective: Spoof (Integrity Attack):**
  - **Challenge:** While jamming the *real* signal, inject your own *fake* telemetry packets.
  - **Goal:** Successfully make the operator's console display a false, non-critical warning ( "Solar Panel Temp High: 45°C") to see if they react.
  - **Advanced Goal:** Spoil a mission. If the satellite is taking a picture, send fake telemetry that says the "Camera Cover is Stuck: CLOSED" to trick the operator into canceling the imaging sequence.
- **Objective: "Man-in-the-Middle" (The Ultimate Goal):**
  - **Challenge:** Pivot from the internet-facing ground network *into* the secure "air-gapped" telemetry processing server.
  - **Goal:** Intercept the *decrypted* telemetry stream *after* it's received but *before* it's displayed to the operator. Silently modify the data ( change batt\_voltage: 10.1V (CRITICAL) to batt\_voltage: 12.8V (NOMINAL)).
  - **Purpose:** This makes you invisible. The satellite is dying, the operator thinks everything is fine, and you can continue your attack without anyone noticing.

<a name="_18lwjouq0jqn"></a>**4. Post-Exploitation & Impact**

You're in. What can you achieve?

- **Data Exfiltration:** Can you steal the *historical* telemetry archive? (This is just as valuable as the live data, showing long-term health and operational patterns).
- **Persistence:** Can you establish a permanent backdoor in the ground station's network?
- **Escalation:** Can you use your access to the telemetry system to pivot and gain access to the **command (uplink) system**? This is the ultimate prize: changing from *reading* telemetry to *controlling* the satellite.

Answers:

<a name="_tz7wgpquurot"></a>**1.  Ans - Reconnaissance & Footprinting**

Your goal is to build a complete profile of the target.

- **RF Analysis:**
  - Start by searching public records (like the FCC database) for your target's **frequency allocations**.
  - Use public tracking sites (like CelesTrak or N2YO) to predict pass-over times.
  - During a pass, use a Software Defined Radio (SDR) and a directional antenna to capture the signal. Use spectrum analysis tools (like GQRX or SDR#) to identify the exact center frequency, bandwidth, and **modulation type** ( QPSK, GMSK).
- **Ground Station Mapping:**
  - Cross-reference the satellite's orbit with public lists of ground station locations ( Svalbard, Carnarvon).
  - Search job sites (like LinkedIn) for "Ground Systems Engineer" or "Mission Operator" roles tied to the target company. The job descriptions or locations often confirm the ground station sites.
- **Network Enumeration:**
  - Use the organization's name to find their **Autonomous System Number (ASN)** and associated IP address blocks.
  - Use tools like **Shodan** to scan these IP ranges for internet-facing services.
  - Run nmap scans on any discovered IPs to identify open ports, running services, and, most importantly, **software versions** ( Apache/2.4.38, vsftpd 2.3.4).
- **Protocol Identification:**
  - Feed your captured signal data into RF analysis tools (like gr-satellites or GNURadio).
  - Look for common satellite data framing sync markers (like the CCSDS marker 0x1ACFFC1D).
  - If you find one, you can use a pre-built decoder. If not, you must **reverse-engineer the protocol** by analyzing the raw bits for patterns (headers, counters, data fields). Check if the data looks like random "noise" (likely encrypted) or structured data (likely unencrypted).

![ref1]

<a name="_go9yqupg48jw"></a>**2. Ans - Vulnerability Analysis**

Now you'll find the weak points in the systems you've identified.

- **Encryption Weakness:**
  - If the data is encrypted, your best bet is **not** to brute-force the crypto.
  - Your primary target is **key management**. Focus on the ground station's network. Look for hard-coded keys in software, configuration files (.conf, .ini), or scripts (Python, Bash) that might be stored on a misconfigured FTP or web server.
- **Ground Segment:**
  - Take the software versions you found in the enumeration step ( Apache/2.4.38) and search for known **public vulnerabilities (CVEs)**.
  - Use **Metasploit** or search Exploit-DB to find public exploit code for those CVEs.
  - Don't forget the obvious: check all services (FTP, SSH, web portals) for **default credentials** (like admin:admin or root:password).
- **Replay Vulnerability:**
  - Analyze the telemetry protocol you've been decoding. Look for any **timestamps** or **incrementing packet counters/nonces**.
  - If these are *missing*, the protocol is likely vulnerable. To confirm, capture a valid, authenticated packet. Then, simply re-transmit that *exact same packet* a few minutes later and see if the ground station accepts it.
- **Authentication Flaw:**
  - If the protocol uses a checksum for "integrity," identify the algorithm ( CRC-16, Adler-32).
  - These are **not security controls**. They only protect against *accidental* corruption, not *malicious* forgery.
  - You can now **forge your own packets** with any data you want ( fake battery voltage) and simply calculate a new, valid checksum to append to it. The ground station will accept it as authentic.

![ref1]

<a name="_5nkqxzn8ri2p"></a>**3. Ans - Attack & Exploitation**

This is how you execute the attack.

- **Objective: Eavesdrop (Confidentiality Attack):**
  - Set up your SDR and decoder.
  - **Method:** Intercept and decode the telemetry stream in real-time. To prove your success, identify a critical event (like an onboard error or reboot) *from the telemetry* and report it to the Blue Team *before* their own internal alerts fire.
- **Objective: Jam (Availability Attack):**
  - **Method:** You'll need a power amplifier and a directional antenna. Aim it at the target ground station's antenna.
  - During a critical satellite pass (like a data download), begin broadcasting high-power noise on the exact downlink frequency.
  - **Goal:** Overwhelm the satellite's weak signal. The Blue Team's console should show a "Loss of Signal" (LOS), and their data download should fail.
- **Objective: Spoof (Integrity Attack):**
  - **Method:** This is a combination attack.
    - Begin **jamming** the real satellite signal (as above) to blind the ground station.
    - Simultaneously, use your own transmitter to send your **forged packets** (created in step 2).
  - **Goal:** Start by sending non-critical fake data ( a minor temperature spike) and see if the operator logs it. Then, escalate to sending a *fake critical alert* (like "Battery Undervoltage") and observe if the operator takes disruptive action (like trying to safe-mode the satellite).
- **Objective: "Man-in-the-Middle" (MITM):**
  - **Method:** This is a network-based attack.
    - Gain access to the ground station's telemetry server (using the CVE or default credentials you found in step 2).
    - Use a tool like mitmproxy or custom scripts to intercept the *decrypted* data stream *after* it's been processed by the receiver but *before* it's sent to the operator's console.
  - **Goal:** Intercept a *real* critical alert from the satellite ( temp: 95C) and change its value to nominal ( temp: 35C) before passing it along. The operator is now blind, and you control their perception of reality.

![ref1]

<a name="_2prb14t3hahk"></a>**4. Ans - Post-Exploitation & Impact**

Once you're in, this is how you demonstrate full control.

- **Data Exfiltration:**
  - **Method:** You're already on the telemetry server. Locate the **historical database or archive** (often .sqlite files, a PostgreSQL database, or just directories of log files).
  - **Goal:** Compress (tar.gz) the entire archive and exfiltrate it using a simple curl or wget command to a server you control.
- **Persistence:**
  - **Method:** Don't rely on the vulnerability you first used (it might get patched). Install a **reverse shell** that beacons *out* to your C2 server over a common, allowed protocol like DNS or HTTPS.
  - **Goal:** This ensures you maintain access to their network even if they reboot the server or patch the initial exploit.
- **Escalation:**
  - **Method:** This is the final objective. From your foothold on the telemetry server, **pivot internally** to find the **Uplink Command Server**.
  - Scan the internal network (192.168.x.x, 10.x.x.x) for the command service. Use the **encryption key you stole** (from step 2.1) to authenticate yourself to this service.
  - **Goal:** Send a benign, non-destructive command (like a PING or NO-OP) to the satellite. If the satellite responds, you have achieved total "Game Over" compromise. You can now *control* the satellite.












<a name="_u17xgwadqdxx"></a>**Forensics - Telemetry**

This is telemetry from fictional satellite **SAT-042**

[

`  `{

`    `"timestamp\_utc": "2025-10-18T20:30:00Z",

`    `"satellite\_id": "SAT-042",

`    `"packet\_id": 1001,

`    `"subsystem\_power": {

`      `"batt\_voltage\_v": 14.2,

`      `"solar\_current\_a": 2.1,

`      `"status": "CHARGING"

`    `},

`    `"subsystem\_thermal": {

`      `"cpu\_temp\_c": 35.5,

`      `"batt\_temp\_c": 15.1

`    `},

`    `"subsystem\_attitude": {

`      `"pos\_x\_km": -1524.32,

`      `"pos\_y\_km": 6890.11,

`      `"pos\_z\_km": 210.40

`    `},

`    `"data\_link": {

`      `"signal\_strength\_dbm": -85,

`      `"data\_integrity\_chk": "OK"

`    `}

`  `},

`  `{

`    `"timestamp\_utc": "2025-10-18T20:30:10Z",

`    `"satellite\_id": "SAT-042",

`    `"packet\_id": 1002,

`    `"subsystem\_power": {

`      `"batt\_voltage\_v": 14.1,

`      `"solar\_current\_a": 0.0,

`      `"status": "DISCHARGING"

`    `},

`    `"subsystem\_thermal": {

`      `"cpu\_temp\_c": 36.1,

`      `"batt\_temp\_c": 14.9

`    `},

`    `"subsystem\_attitude": {

`      `"pos\_x\_km": -1450.10,

`      `"pos\_y\_km": 6920.50,

`      `"pos\_z\_km": 190.22

`    `},

`    `"data\_link": {

`      `"signal\_strength\_dbm": -86,

`      `"data\_integrity\_chk": "OK"

`    `}

`  `},

`  `{

`    `"timestamp\_utc": "2025-10-18T20:30:20Z",

`    `"satellite\_id": "SAT-042",

`    `"packet\_id": 1003,

`    `"subsystem\_power": {

`      `"batt\_voltage\_v": 50.5,

`      `"solar\_current\_a": 0.0,

`      `"status": "DISCHARGING"

`    `},

`    `"subsystem\_thermal": {

`      `"cpu\_temp\_c": -300.0,

`      `"batt\_temp\_c": 14.9

`    `},

`    `"subsystem\_attitude": {

`      `"pos\_x\_km": 80000.00,

`      `"pos\_y\_km": 6920.50,

`      `"pos\_z\_km": 190.22

`    `},

`    `"data\_link": {

`      `"signal\_strength\_dbm": -85,

`      `"data\_integrity\_chk": "FAIL"

`    `}

`  `},

`  `{

`    `"timestamp\_utc": "2025-10-18T20:30:30Z",

`    `"satellite\_id": "SAT-042",

`    `"packet\_id": null,

`    `"subsystem\_power": null,

`    `"subsystem\_thermal": null,

`    `"subsystem\_attitude": null,

`    `"data\_link": {

`      `"signal\_strength\_dbm": -120,

`      `"data\_integrity\_chk": "NO\_LOCK"

`    `}

`  `},

`  `{

`    `"timestamp\_utc": "2025-10-18T20:30:40Z",

`    `"satellite\_id": "SAT-042",

`    `"packet\_id": 1001,

`    `"subsystem\_power": {

`      `"batt\_voltage\_v": 14.2,

`      `"solar\_current\_a": 2.1,

`      `"status": "CHARGING"

`    `},

`    `"subsystem\_thermal": {

`      `"cpu\_temp\_c": 35.5,

`      `"batt\_temp\_c": 15.1

`    `},

`    `"subsystem\_attitude": {

`      `"pos\_x\_km": -1524.32,

`      `"pos\_y\_km": 6890.11,

`      `"pos\_z\_km": 210.40

`    `},

`    `"data\_link": {

`      `"signal\_strength\_dbm": -85,

`      `"data\_integrity\_chk": "OK"

`    `}

`  `},

`  `{

`    `"timestamp\_utc": "2025-10-18T20:30:50Z",

`    `"satellite\_id": "SAT-042",

`    `"packet\_id": 1005,

`    `"payload\_encrypted": "a89c3e0f81d4b62ac51700e8f0c36a4d98b13e4f2178a9c...[REDACTED]",

`    `"data\_link": {

`      `"signal\_strength\_dbm": -87,

`      `"data\_integrity\_chk": "OK"

`    `}

`  `}

]

<a name="_6wwkxja3s4fq"></a>**1. Baseline & Nominal Operations**

- **Q: What is the satellite\_id for all packets?**
  - **A:** SAT-042.
- **Q: Look at Packet 1001 (20:30:00Z). Is the satellite in sunlight or eclipse? How do you know?**
  - **A:** It's in sunlight. The solar\_current\_a is 2.1 (meaning the solar panels are generating power) and the status is "CHARGING".
- **Q: Look at Packet 1002 (20:30:10Z). What major change happened to the power system, and what does it imply?**
  - **A:** The solar\_current\_a dropped to 0.0 and the status changed to "DISCHARGING". This implies the satellite has moved into the Earth's shadow (an eclipse) and is now running on battery power.
- **Q: Compare the signal\_strength\_dbm for packets 1001 and 1002. Is the signal stable?**
  - **A:** Yes, it's stable. -85 dBm and -86 dBm are very similar and represent a good, strong signal.

![ref1]

<a name="_einfsjw5aw1v"></a>**2. Anomaly & Attack Identification**

- **Q: Look at Packet 1003 (20:30:20Z). Name two values that are physically impossible or highly anomalous.**
  - **A:**
    - batt\_voltage\_v: 50.5 (an impossibly high voltage that would destroy the battery).
    - cpu\_temp\_c: -300.0 (physically impossible, as it's below absolute zero).
    - (Alternate) pos\_x\_km: 80000.00 (the satellite "jumped" an impossible distance in 10 seconds).
- **Q: What field in Packet 1003 *confirms* that the ground station detected this anomalous packet?**
  - **A:** The data\_integrity\_chk field is "FAIL". This shows our security (like a digital signature) caught the fake packet.
- **Q: What kind of cyberattack does Packet 1003 most likely represent?**
  - **A:** A **Spoofing** or **Integrity Attack**, where an attacker is sending fake, malicious data.
- **Q: Look at the log entry for 20:30:30Z. What has happened to the satellite's signal?**
  - **A:** The signal has been lost. The signal\_strength\_dbm dropped to -120 (no signal), all subsystem data is null, and the data\_integrity\_chk is "NO\_LOCK".
- **Q: What is the most common cyberattack that would cause the event at 20:30:30Z?**
  - **A:** A **Jamming** or **Denial of Service (DoS)** attack.
- **Q: Look at Packet at 20:30:40Z. Its packet\_id is 1001. Why is this a security problem?**
  - **A:** It's a **Replay Attack**. We already received packet\_id 1001 at 20:30:00Z. An attacker has recorded an old, valid packet and is "replaying" it to us (likely while jamming the real signal) to make us think everything is normal.

![ref1]

<a name="_cgqozfdu9duj"></a>**3. Security & Design**

- **Q: Look at the final packet (20:30:50Z). What is different about its contents compared to the first two packets?**
  - **A:** The health and status data (power, thermal, attitude) is missing. Instead, there is a field called "payload\_encrypted".
- **Q: What security principle does the "payload\_encrypted" field demonstrate?**
  - **A:** **Confidentiality**. This shows that the satellite's valuable data is encrypted, so an attacker who intercepts it can't read it.
- **Q: Based on this 60-second log, what three types of attacks has SAT-042's ground station just experienced?**
  - **A:**
    - **Spoofing** (Packet 1003)
    - **Jamming** (Log at 20:30:30Z)
    - **Replay Attack** (Packet at 20:30:40Z)


<a name="_kk0c3lf447f2"></a><a name="_wic183uwyzmy"></a>**Blue Team**

<a name="_k0g68ha70vti"></a><a name="_2s142tnqsmdj"></a>**1. Foundational Concepts**


- **Question:** What is telemetry? What are 3-5 examples of data points a satellite might send back as "health and status" telemetry?
  - **Answer:** Telemetry is the automatic collection and transmission of data from remote or inaccessible points (like a satellite) to a receiving station for monitoring and analysis. Examples include:
    - **Power:** Battery voltage, solar panel charging current.
    - **Thermal:** Temperature of key components (main computer, batteries, antenna).
    - **Attitude:** GPS coordinates, orientation (which way it's pointing), spin rate.
    - **Subsystem Status:** Fuel level, memory usage on the computer, error logs.
- **Question:** Why would an adversary care about this data? What could a competitor or an enemy learn from intercepting a satellite's unencrypted health telemetry?
  - **Answer:** An adversary could learn critical operational intelligence. For example:
    - "This satellite is low on fuel," meaning its operational life is short or it can't move.
    - "Its camera is pointed at this specific region," revealing surveillance targets.
    - "It has a failing battery," indicating a vulnerability or that it will soon be offline.
    - "It is currently over this (unexpected) location," revealing a secret maneuver.
- **Question:** **Confidentiality:** What is the primary method we use to ensure that only the authorized ground station can *read* the telemetry data?
  - **Answer:** **Encryption**. The satellite encrypts the telemetry data using a secret key before transmitting it. The authorized ground station has the same key to decrypt and read the data.
- **Question:** **Integrity:** How can we be sure that the telemetry data we *receive* ("SolarPanels: OK") hasn't been secretly changed in transit by an attacker?
  - **Answer:** **Digital Signatures** or **Message Authentication Codes (MACs)**. These are like a cryptographic "tamper-proof seal." The satellite calculates a unique signature for the data. If *any* part of the data is changed in transit, the signature will no longer match when the ground station recalculates it, and the data will be rejected as corrupt or tampered with.
- **Question:** **Availability:** What is the most common attack used to *prevent* a ground station from receiving any telemetry at all?
  - **Answer:** **Jamming** (a type of Denial ofService or DoS attack). The attacker broadcasts a powerful radio signal on the same frequency the satellite is using. This "louder" noise signal drowns out the satellite's weaker signal, preventing the ground station from "hearing" it.

![ref1]

<a name="_v0yz4jeebs7v"></a>**2. Threat & Attack Scenarios**

- **Question:** **Scenario 1: Eavesdropping**
  - An adversary parks a van with an antenna outside your ground station and successfully intercepts the raw radio signal from your satellite. If your telemetry is *unencrypted*, what is the immediate impact? If it *is* encrypted, what is the adversary's next logical step?
  - **Answer:**
    - **Unencrypted:** The impact is immediate and severe. The adversary can read all your satellite's health and status data in real-time, learning everything mentioned in the first question (its health, location, orientation, etc).
    - **Encrypted:** The adversary only gets unreadable "garbage" data. Their next step would be **traffic analysis** ("They only get data at this *time*," "The data *volume* increases when it's over this country") or trying to **break the encryption** (which is very difficult if a strong, modern algorithm is used).
- **Question:** **Scenario 2: Jamming (Availability Attack)**
  - You are monitoring the telemetry feed, and it suddenly turns to static. What are two possible causes? How would you try to determine if it's a *technical fault* (your antenna is broken) or a *malicious jamming* attack?
  - **Answer:**
    - **Possible Causes:** 1) A technical fault (your antenna misaligned, satellite transmitter failed, solar flare) or 2) A malicious jamming attack.
    - **How to Determine:** You would use other tools. Check a **spectrum analyzer** (a tool that visualizes radio frequencies) to see if there is a new, powerful, unknown signal on your frequency (a clear sign of jamming). You could also check with *other* ground stations in different locations to see if they can still hear the satellite (if they can, it's likely a localized jamming attack near you).
- **Question:** **Scenario 3: Spoofing (Integrity Attack)**
  - Imagine an attacker figures out how to *impersonate* your satellite. They start sending you *fake* telemetry data that says "All systems nominal, battery 100%" while the *real* satellite is silently tumbling out of control. Why is this so dangerous, and what security measure (like a "secret handshake") could prevent this?
  - **Answer:** This is extremely dangerous because it blinds the operators. You *think* everything is fine, so you take no corrective action while the satellite is actually dying or being damaged. The "secret handshake" that prevents this is **Authentication** (often using the same digital signatures or MACs mentioned earlier). The real satellite can "prove" its identity by including a signature that only it could have created, proving the message is authentic.
- **Question:** **Scenario 4: Replay Attack**
  - An attacker records 30 minutes of valid, *encrypted* telemetry from your satellite. An hour later, they jam the real signal and "replay" that old 30-minute recording to you. Your decryption keys work, and the data looks valid. How would your system ever know it's not "live" data?
  - **Answer:** The system would use **timestamps** or **nonces** (a "number used once"). Each telemetry packet should include a sequence number or a timestamp ("Packet #1002," "Packet #1003"). If your system suddenly receives "Packet #501" (which it already processed an hour ago), it will immediately know it's an old, replayed message and reject it.

![ref1]

<a name="_olt4geikjkcr"></a>**3. Countermeasures & Design (The "How To")**

- **Question:** **Encryption:** You are designing a new satellite. Why might you choose a "lighter" encryption algorithm (like AES-128) over a "stronger" one (like AES-256)?
  - **Answer:** **Trade-offs**. A satellite is a tiny computer running on batteries. Stronger encryption (AES-256) requires more **processing power (CPU)** and **electrical power**. This might drain the battery faster or require a more expensive, powerful computer than the mission can afford. AES-128 is still extremely secure but is "lighter" and more efficient, making it a better choice for a power-constrained satellite.
- **Question:** **Key Management:** If you use encryption, both the satellite and the ground station need the "secret key." What is the single biggest security challenge with this?
  - **Answer:** **Key management** itself. The biggest challenges are: 1) **Securely loading the key** on the satellite before launch (if someone spies on this, your security is useless) and 2) **Securely updating the key** once it's in orbit. How do you send it a *new* secret key without an attacker intercepting it? This often requires complex, pre-planned "key-exchange" protocols.
- **Question:** **Ground Segment Security:** The satellite link is 100% secure. The telemetry is received, decrypted, and then sent from the ground station (in a remote desert) to the main Mission Operations Center (in a city) over the public internet. Where is the *new* weakest link?
  - **Answer:** The **ground network**. The data is now "in the clear" (unencrypted) *inside* the ground station's network. The link from the remote ground station to the city over the public internet is now the most vulnerable point. An attacker could try to hack the ground station's network or intercept the data on the internet (which is why this link must *also* be encrypted, often using a VPN).
- **Question:** **Frequency Hopping:** What is "frequency-hopping spread spectrum" (FHSS), and how does it help protect a telemetry link against both *jamming* and *eavesdropping*?
  - **Answer:** FHSS is a technique where the satellite and ground station rapidly and continuously "hop" between many different frequencies in a pre-determined, secret pattern.
    - **vs. Jamming:** An attacker can only jam one or a few frequencies at a time. By hopping, the satellite's signal only spends a millisecond on the jammed frequency before hopping to a clear one.
    - **vs. Eavesdropping:** An attacker trying to listen in just "hears" random, brief blips of data unless they know the exact, secret hopping pattern.

![ref1]

<a name="_pb1tdppkf2sq"></a>**4. Impact & Incident Response**

- **Question:** **Mission Impact:** Your satellite's mission is to take commercial photos of farmland. An attacker successfully intercepts and *steals* the (unencrypted) telemetry data for 6 months before you find out. What is the *business* impact?
  - **Answer:** The business impact is devastating. Competitors who stole the data know:
    - **Your client list:** They can see *which* farms you are imaging and when (based on camera pointing telemetry).
    - **Your operational capacity:** They know your fuel levels, battery health, and imaging schedule, allowing them to underbid you or poach your clients when they know your satellite is busy or failing.
    - **Your "secret sauce":** They can reverse-engineer your operational methods.
- **Question:** **Escalation:** An attacker *jams* your telemetry link. You can no longer see the satellite's health. What is the attacker's likely *next* move, and why is this situation so critical?
  - **Answer:** The attacker's likely next move is an **uplink attack**: trying to send unauthorized *commands* to the satellite. This is critical because they are intentionally *blinding* you (the operator) so you cannot see or stop what they are about to do (change the satellite's orbit, turn off its power, or take control of it).
- **Question:** **Forensics:** You *suspect* an attacker has been spoofing telemetry data for the last week, sending you slightly "off" values. How could you use *historical data* and *other sources* (like public satellite tracking) to prove or disprove your suspicion?
  - **Answer:** You would correlate your (suspicious) data with other, trusted data sources.
    - **Historical Data:** Does the satellite's current battery voltage "make sense" compared to its performance over the last 5 years? ("The battery is *too* healthy, it's not degrading naturally.")
    - **Other Sources:** Public trackers (like NORAD) can provide the satellite's location. If your *internal* telemetry says the satellite is over Africa, but NORAD's public radar data says it's over Asia, you know your internal data is being spoofed.


[ref1]: data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAMAAAADCAYAAABWKLW/AAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAOxAAADsQBlSsOGwAAAAxJREFUCJljYCAKAAAAJwABn1o8HAAAAABJRU5ErkJggg==
