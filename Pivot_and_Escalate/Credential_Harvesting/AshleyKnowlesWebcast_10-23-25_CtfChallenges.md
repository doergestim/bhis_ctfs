### []{#anchor}Question 1: (Scoping & PTES) {#question-1-scoping-ptes}

**Scenario:** A client sends you a signed contract that defines the scope as: *\"All assets associated with our primary domain, \'example.com\'.\"*

During your initial OSINT (Open-Source Intelligence) phase, you discover three servers:

- *www.example.com* (IP: *1.2.3.4*)
- *mail.example.com* (IP: *1.LET.3.5*)
- *dev-portal.example.com* (IP: *1.2.3.6*), which appears to be a login for a third-party SaaS provider (like Atlassian or Salesforce).

**Question:** According to the \"Pre-engagement Interactions\" phase of the Penetration Testing Execution Standard (PTES), what is your *immediate next step*?

**Answer:** Your immediate next step is to **stop and ask the client for clarification**.

The vague scope *\"all assets associated with \'example.com\'\"* is a trap.

1.  The mail server (*mail.example.com*) might be managed by a third party (like Google Workspace or Microsoft 365), and attacking it would be illegal and violate the provider\'s terms of service.
2.  The *dev-portal.example.com* is explicitly a third-party provider. Attacking it is **out of scope** and illegal.
3.  The *only* asset you can be reasonably sure is in scope is *www.example.com*.

A professional tester must get **explicit, written permission** (ideally a list of IP addresses or ranges) from the client *before* launching any active scans or tests. This protects both you and the client from legal issues and accidental outages.

### []{#anchor-1}Question 2: (OSINT Techniques)

**Scenario:** During your OSINT phase, you are researching a company\'s developers on GitHub. You find a public repository from a developer who works at the target company. In the \"commit history\" for a file named *config.py*, you find this:

\- \# db_pass = \"supersecurepassword!123\"

\- \# db_user = \"admin\"

\- \# db_ip = \"192.168.1.50\"

\+ db_pass = os.environ.get(\"DB_PASS\")

\+ db_user = os.environ.get(\"DB_USER\")

\+ db_ip = os.environ.get(\"DB_IP\")

**Question:** What have you found, and what is the *critical distinction* between documenting this finding and acting on it?

**Answer:** You have found **hard-coded credentials** (a password, username, and IP address) in the project\'s commit history. The developer accidentally committed them and then later \"removed\" them, not realizing they are saved in the project\'s history forever.

- **What to do (Document):** You must immediately **document** these credentials in your notes. The IP *192.168.1.50* is an *internal* IP, suggesting this is the internal database. The credentials *admin* / *supersecurepassword!123* are highly valuable.
- **What NOT to do (Act):** You **cannot** act on this *yet*. You are still in the OSINT phase, which is *passive*. Attempting to connect to any company asset using these credentials would be an **active attack**.
- **How to use it:** You will save these credentials for the **Enumeration & Exploitation** phase. Once you have gained an initial foothold on the *internal* network (which is in scope), you will then try these credentials against the database at *192.168.1.50* to see if they are still valid. This is a classic OSINT-to-internal-pivot.

### []{#anchor-2}Question 3: (Vulnerability Scanners)

**Scenario:** You run a Nessus (vulnerability scanner) scan against an in-scope web server. The scanner reports a **\"Critical\"** vulnerability: *MS15-034: HTTP.sys Remote Code Execution*. You are excited because this is a major finding. However, when you try to use the public exploit script for MS15-034, it fails.

**Question:** What is the most likely reason the exploit failed, and how do you responsibly report this finding?

**Answer:** This is a classic **False Positive**. The vulnerability scanner *did not* actually exploit the server.

- **What happened:** The scanner likely only performed a **version check**. It saw the server was running a version of Windows Server (e.g., IIS 8.5) that *is known to be* vulnerable to MS15-034. It flagged this as \"Critical\" based on the *potential* impact.

- **Why it failed:** The exploit likely failed because the server has a **Web Application Firewall (WAF)** in front of it that blocked your malicious request, *or* the server has been patched, but the scanner couldn\'t detect the patch correctly.

- **How to report:** You **do not** report this as a \"Critical\" RCE. You report it as a \"Low\" or \"Informational\" finding.

  - **Finding:** \"Potential Vulnerability: MS15-034 (Un-validated)\"
  - **Description:** \"The vulnerability scanner identified the server as potentially vulnerable to MS15-034. However, all manual attempts to validate and exploit this vulnerability were unsuccessful, likely due to a compensating control such as a Web Application Firewall (WAF). This indicates the server is not directly exploitable, but the underlying software may still be unpatched.\"

### []{#anchor-3}Question 4: (Exploit vs. Document) {#question-4-exploit-vs.-document}

**Scenario:** You\'re testing a web application and find a function where you can upload a profile picture. You try uploading a web shell (e.g., *shell.php*) but the server rejects it, saying \"Invalid file type. Only JPG or PNG.\"

- You try renaming it: *shell.php.jpg* (Rejected)
- You try a null-byte: *shell.php%00.jpg* (Rejected)
- You try changing the *Content-Type* header (Rejected)
- You try embedding a shell inside a real JPG\'s metadata (Rejected)

You have now spent **four hours** trying to bypass this single file upload.

**Question:** Are you in a \"rabbit hole,\" and what is the productive, professional alternative to continuing this attack?

**Answer:** **Yes, you are in a time-wasting rabbit hole.** While this *might* be exploitable, the time-to-value ratio is now extremely low. The client is paying for a comprehensive test, not for you to spend half a day on one medium-risk function.

The professional alternative is to **pivot and document**.

1.  **Stop** trying to exploit it.
2.  **Document** the finding as-is: \"The file upload function appears to have basic security controls (like file extension and magic byte checks) that successfully prevented simple web shell uploads. However, the *best practice* is to not allow file uploads to be saved in a web-accessible, executable directory at all. All uploads should be saved to a non-web-root directory (e.g., */var/uploads/*) or an S3 bucket with a randomized file name.\"
3.  **Move on** to other, potentially more fruitful areas of the application (like authentication, session management, or other business logic flaws). You can always circle back if you have extra time at the end of the engagement.

### []{#anchor-4}Question 5: The Log Analysis (Telemetry Interpretation)

**Scenario:** A Blue Team analyst is watching the firewall logs during your penetration test. They see the following entries from your testing IP address, all targeting their web server *10.10.1.5*:

|           |             |        |     |                |
|-----------|-------------|--------|-----|----------------|
| *YOUR_IP* | *10.10.1.5* | *80*   | TCP | *\[SYN\]*      |
| *YOUR_IP* | *10.10.1.5* | *21*   | TCP | *\[RST, ACK\]* |
| *YOUR_IP* | *10.10.1.5* | *22*   | TCP | *\[RST, ACK\]* |
| *YOUR_IP* | *10.10.1.5* | *23*   | TCP | *\[RST, ACK\]* |
| *YOUR_IP* | *10.10.1.5* | *443*  | TCP | *\[SYN, ACK\]* |
| *YOUR_IP* | *10.10.1.5* | *3389* | TCP | *\[RST, ACK\]* |

**Question:** What *specific attack tactic* are you performing, and what have you learned from these 6 log entries?

Answer:

- **Attack Tactic:** This is a **TCP SYN Scan** (also known as a \"half-open\" scan), a common technique used by the tool **Nmap**. You are in the \"Active Enumeration\" or \"Scanning\" phase of your test.

- **What you learned:**

  - **Ports 80 (HTTP) and 443 (HTTPS) are OPEN.** The server responded with a *\[SYN, ACK\]*, completing the first two steps of the three-way handshake.
  - **Ports 21 (FTP), 22 (SSH), 23 (Telnet), and 3389 (RDP) are CLOSED.** The server responded with a *\[RST, ACK\]*, indicating that the port is closed and no service is listening.

The analyst is seeing a classic port scan, and your next step would be to enumerate the web services running on ports 80 and 443.

### []{#anchor-5}Question 6: (Reporting Best Practices)

**Scenario:** During an internal network test, you discover a file share (SMB) that is open to the *Everyone* group. Inside, you find 500 GB of data. You spot-check a few files and find they are non-sensitive, like company marketing materials, stock photos, and old press releases. You don\'t find any passwords, PII, or trade secrets.

**Question:** You did not find any \"critical\" data, so how do you report this finding? Is it just \"Informational\"?

**Answer:** This is **not** an \"Informational\" finding; it\'s a \"Medium\" or \"High\" risk finding.

The mistake is focusing only on *what you found* instead of *what this vulnerability allows*. The finding isn\'t \"Old marketing files were found.\" The finding is:

- **Finding:** \"Insecure File Share Permissions Allow Uncontrolled Data Access\"
- **Risk:** Medium/High
- **Description:** \"A network file share (e.g., *\\SERVER01\Share*) was configured with *Everyone* permissions. This is a critical data hygiene and access control failure. While only non-sensitive data was observed during the test, an attacker or malicious insider could use this share to **exfiltrate data** undetected or, more dangerously, **plant malware** in a location trusted by other employees.\"
- **Recommendation:** \"Apply the principle of least privilege. Remove the *Everyone* group and grant access only to specific Active Directory groups that require it.\"

This demonstrates how a pentester reports on *potential risk* and bad practice, not just on the successful exploitation of data.

### []{#anchor-6}Question 7: (NIST CSF)

**Scenario:** A client is very focused on the **NIST Cybersecurity Framework (CSF)**. After your pentest, they ask, \"How do your findings help us with the \'Identify\' (ID) and \'Protect\' (PR) functions of the CSF?\"

**Question:** How do you answer them?

**Answer:** You explain how the pentest directly validates (or finds gaps in) those specific functions:

- **For \"Identify\" (ID):** \"The *Identify* function is about knowing what you have. Our scans and enumeration (*Nmap*, etc.) are a real-world test of your **Asset Management (ID.AM)**. We found three servers that your IT team was unaware of. This shows a gap in your *ID.AM* controls, as you can\'t protect what you don\'t know you have.\"
- **For \"Protect\" (PR):** \"The *Protect* function is about implementing safeguards. Our report is a direct audit of your **Access Control (PR.AC)**. For example, our finding that we could access the finance share from a guest-level account (Finding 4.1) is a clear failure of your *PR.AC-4* (Information Access Control) policies. Our recommendations are a direct roadmap to strengthening this control.\"

This maps your technical findings to their business and compliance-oriented framework, making the report far more valuable.

### []{#anchor-7}Question 8: (Avoiding Rabbit Holes)

**Scenario:** You find a valid set of credentials for an employee (*jsmith* / *Winter2024!*) through OSINT. You try to log in to the company\'s Outlook Web Access (OWA) portal, but it fails. You try the VPN, and it also fails (it likely requires Multi-Factor Authentication). You\'ve now spent 30 minutes trying to use these credentials externally.

**Question:** Is this a rabbit hole? What is the *next logical step*?

**Answer:** This is only a rabbit hole if you *keep trying* to use them against external, MFA-protected services. The credentials themselves are still **gold**.

The next logical step is to **pivot your attack vector**.

1.  **Stop** trying to use them externally.
2.  **Save** them in your notes.
3.  **Change focus:** Your new goal is to find a *different, less-secure* service that might be using the same credentials (a tactic called **credential stuffing**).
4.  **Scan** the company\'s IP ranges for other, non-obvious services. You might find a forgotten WordPress site (*blog.example.com*), an old FTP server (*ftp.example.com*), or a remote desktop port (*3389*).
5.  **Try** the *jsmith* credentials against *those* services. It\'s highly likely the old, forgotten FTP server doesn\'t have MFA and *will* let you in, giving you your first foothold.

### []{#anchor-8}Question 9: (Telemetry Interpretation)

**Scenario:** You\'ve moved past the initial port scan (Question 5) and are now running a *service version detection* scan with Nmap (*-sV*). The Blue Team sees this traffic in their logs from your IP:

|              |                |           |          |                                                                                     |
|--------------|----------------|-----------|----------|-------------------------------------------------------------------------------------|
| Source IP    | Destination IP | Dest Port | Protocol | Packet Info / Payload                                                               |
| *YOUR_IP*    | *10.10.1.10*   | *22*      | TCP      | *\[PSH, ACK\]* Payload: *SSH-2.0-MyTestClient\r\n*                                  |
| *10.10.1.10* | *YOUR_IP*      | *22*      | TCP      | *\[PSH, ACK\]* Payload: *SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.1\r\n*               |
| *YOUR_IP*    | *10.10.1.10*   | *80*      | TCP      | *\[PSH, ACK\]* Payload: *GET / HTTP/1.1\r\nHost: 10.10.1.10\r\n\r\n*                |
| *10.10.1.10* | *YOUR_IP*      | *80*      | TCP      | *\[PSH, ACK\]* Payload: *HTTP/1.1 200 OK\r\nServer: Apache/2.4.41 (Ubuntu)\r\n\...* |

**Question:** What are you doing *specifically*, and what two critical pieces of information have you learned from the *responses*?

**Answer:** You are performing **Service Version Enumeration**. Unlike a simple SYN scan (which just checks if a port is open), you are now actively *talking* to the services on the open ports to make them identify themselves.

The two critical pieces of information you\'ve learned from the server\'s responses are:

1.  **Port 22:** The server is running **OpenSSH version 8.2p1** on **Ubuntu**. This is extremely valuable. You can now search for specific exploits for *OpenSSH 8.2p1* (e.g., *cve-2020-15778*, a username enumeration flaw).
2.  **Port 80:** The server is running **Apache version 2.4.41** on **Ubuntu**. Again, you can now search for specific exploits, vulnerable modules (*mod_x*), or misconfiguration defaults for this *exact* version of Apache.

This is a much \"louder\" scan, but it provides the precise data needed for the exploitation phase.
