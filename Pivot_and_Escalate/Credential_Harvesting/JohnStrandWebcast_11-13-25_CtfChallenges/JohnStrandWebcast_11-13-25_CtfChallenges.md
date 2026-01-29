**Question 1: The "Which-is-Worse" Scenario**

You get two high-fidelity alerts at the same time:

- **Alert A:** mimikatz.exe blocked by EDR on a workstation.
- **Alert B:** powershell.exe on a Domain Controller runs a command to create a new system task.

Based on the Volt Typhoon model, which alert is **more critical** and why?

**Answer:** **Alert B is far more critical.** Alert A is a "win" for defense; a known-bad tool was caught and blocked. That's *old-school* (ransomware, script kiddies).

Alert B is classic Volt Typhoon. They **"live off the land"** (LotL). They use *your own tools* (PowerShell) on a critical server (Domain Controller) to do something *that an admin might do* (create a task) for persistence. This blends in, avoids EDR signatures, and is the "silent killer" of pre-positioning.

![ref1]

**Question 2: The "Wrong Target" Scenario**

An attacker has breached your network. Your logs show them trying to move laterally. Which of these two jump attempts is the *classic signature* of a Volt Typhoon-style actor?

- **Move A:** The attacker is exfiltrating the entire database from the **HR file server**.
- **Move B:** The attacker is trying to get a shell on the **HVAC controller's management workstation**.

**Answer:** **Move B (the HVAC controller).** Stealing HR data (Move A) is espionage (like Salt Typhoon) or theft. It's about *data*.

Volt Typhoon is about **disruption**. They are pre-positioning in critical infrastructure. Gaining access to the HVAC, the power grid, or the water controls is their entire goal. They're not there to steal; they're there to plant a "digital bomb" and wait.

![ref1]





**Question 3: The "Netsh" Puzzle**

You find this command run on a compromised server. What is it *doing*, and why is it a favorite Volt Typhoon TTP?

netsh interface portproxy add v4tov4 listenport=3389 listenaddress=0.0.0.0 connectport=3389 connectaddress=10.0.0.100

**Answer:** It's a **port proxy** (or port forwarding) using the built-in Windows netsh tool. It's telling the compromised server to listen on its *own* RDP port (3389) and *forward* any connection it receives to a *different* machine (10.0.0.100).

This is a **classic LotL pivot technique**. The attacker doesn't have to attack the final target directly. They "bounce" through the compromised server. It's stealthy, uses a legitimate Windows tool, and is incredibly hard to spot in firewall logs because the traffic looks like it's just going to the first server.

![ref1]

**Question 4: The "Parent/Child" Anomaly**

Your EDR is set to "log only" mode. You see this process chain. What's wrong with this picture?

- **Parent:** w3wp.exe (IIS Web Server)
- **Child:** cmd.exe (Command Prompt)
- **Grandchild:** powershell.exe -e <long\_base64\_string>

**Answer:** Everything. This is a **parent/child process anomaly**. A web server (w3wp.exe) should *never* spawn a command shell. Its job is to serve web pages.

This chain shows that an attacker likely exploited a vulnerability in the web application, which gave them the ability to run code. That code (cmd.exe) then launched PowerShell to run an encoded command (the Base64 string), probably to download a payload or connect to a C2 server. This is a *behavioral* giveaway, even if all the tools are "legitimate."

![ref1]





**Question 5: The "Nowhere to be Found" C2**

You're *convinced* a critical infrastructure network is compromised by Volt Typhoon. But:

- Your EDR is silent.
- Your SIEM shows no weird internal traffic.
- All servers are patched.

Where is the *one* place the attacker is most likely "living" and running their Command & Control (C2) from, which *none* of your expensive tools can see?

**Answer:** The **edge appliance**. Specifically, a **SOHO router, firewall, or VPN device**.

This is the *defining* TTP of Volt Typhoon. They compromise the firmware of these "black box" devices that nobody monitors or patches. They live *inside the router*. All their C2 traffic happens on the "internet" side of the device, so your internal tools *never see it*. Any traffic they send *into* your network looks like it's coming from a "trusted" internal router IP. It's the perfect, invisible beachhead.

![ref1]

**Question 6: The "Waiting Game"**

You discover a state-backed group has been in your network for **200 days**. They've compromised admin credentials. But your investigation shows they **haven't stolen any data, encrypted anything, or caused any damage.**

Is this a (A) failed attack, or (B) something else? Explain.

**Answer:** **(B) Something else.** This is the *most terrifying scenario* and the entire point of the webcast.

A ransomware group would have encrypted you on Day 1. An espionage group (Salt Typhoon) would have exfiltrated your data by Day 2.

This group (Volt Typhoon) is different. Their mission is **not** to steal or break things *today*. Their mission is **long-term, persistent, pre-positioning**. They are "camping" in your network, mapping your critical systems, and *waiting* for the geopolitical order. The lack of "smash and grab" *is* the red flag. They're not a burglar; they're a saboteur waiting for the signal.

![ref1]




**Question 7: The "Fileless" Persistence Puzzle**

You're running a deep hunt for persistence on a Domain Controller. You're not finding any malicious files, no new services, and no suspicious scheduled tasks.

However, you *do* find a **WMI Event Subscription**. It's set to trigger On:UserLogon. The "Consumer" part of the subscription is a single, obfuscated line of PowerShell that uses IEX(New-Object Net.WebClient).DownloadString(...)

Why is this a *far more advanced* persistence technique, and why is it a perfect fit for a Volt Typhoon-style actor?

**Answer:** This is a **fileless persistence mechanism**.

1. **It's "Fileless":** The entire malicious payload (the script) "lives" inside the WMI database (objects.data), not as a .ps1 or .exe file on the disk. This makes it invisible to traditional antivirus scanners that look for "bad files."
1. **It's "LotL":** It uses 100% legitimate, built-in Windows components: WMI (for triggering) and PowerShell (for execution). No foreign binaries are involved.
1. **It's "Stealthy":** It's event-driven. The malicious code *only* runs when a user logs in, making its execution blend in with the "noise" of a normal logon. It's not a service that's "always on" and easy to spot.

This is the *epitome* of the Volt Typhoon mindset: hide in plain sight, use the system's own tools against it, and leave no trace on the file system for simple tools to find.

![ref1]










**Question 8: The "Impossible" Pivot**

You are hunting on your network and analyzing **Netflow logs**. You *cannot* see inside your SOHO routers and firewalls (where Volt Typhoon is known to live).

You see the following log entries. What is this pattern *proving*, and why is it your "smoking gun"?

|Timestamp|Source IP|Dest IP|Dest Port|
| :- | :- | :- | :- |
|14:32:01|1\.2.3.4|YOUR\_FIREWALL|8443|
|14:32:05|YOUR\_FIREWALL|10\.1.1.5 (DC\_01)|445|
|14:32:07|YOUR\_FIREWALL|10\.1.1.6 (FILE\_SVR)|445|
|14:32:10|1\.2.3.4|YOUR\_FIREWALL|8443|

*(Context: 1.2.3.4 is an unknown IP on the internet. YOUR\_FIREWALL is your firewall's internal IP, 10.0.0.1)*

**Answer:** This pattern proves the **firewall itself is compromised and is being used as a pivot point.**

Your firewall (10.0.0.1) should *route* traffic, not *initiate* it.

1. The attacker (1.2.3.4) connects to a C2 listener *on the firewall* (port 8443).
1. The compromised firewall *itself* then initiates **new, internal connections** (on port 445 - SMB) to your Domain Controller and File Server to probe them.

Your internal servers will only see a "trusted" connection coming *from the firewall's IP*. They have no idea it's the attacker. This Netflow log is the *only* place you'd see this "impossible" behavior. This is the *how* they "live on the edge" and pivot inland.

![ref1]




**Question 9: The "PoliSci" Dilemma**

You are a lead hunter at a company that runs **both a major R&D division (developing new tech) and a public water utility.** You find two, separate, *equally stealthy* attackers in your network. You only have the resources to fully hunt and eradicate *one* of them right now.

- **Attacker A (Salt Typhoon):** Is deep in your **R&D network**. Logs show them slowly exfiltrating small, encrypted archives of design documents.
- **Attacker B (Volt Typhoon):** Is deep in your **water utility SCADA network**. Logs show them *only* mapping the network and accessing control system manuals. They have stolen **zero** data.

Which attacker poses the more **urgent, existential threat** *according to the premise of the webcast*, and why is the *lack* of data theft the scariest part?

**Answer:** **Attacker B (the one in the SCADA network) is the more urgent threat.**

- **Attacker A (Salt Typhoon)** is committing **espionage**. They are *stealing* your tech. This is bad, but it's a known, "traditional" state-backed threat. It's about economic gain.
- **Attacker B (Volt Typhoon)** is **pre-positioning for disruption**. The *reason* they haven't stolen data is that data is *not their goal*. Their goal is to learn how to **turn off the water**. The "mapping" and "reading manuals" is them doing their homework, "planting the charges" for a future conflict.

The lack of data theft signifies a motive that is *purely* destructive. They aren't there to rob you; they're there to wait for the signal to burn the house down. This is the "frightening" new threat the webcast is all about.

[ref1]: Aspose.Words.d416e552-bfc5-4123-809d-27a7e6aec835.001.png
