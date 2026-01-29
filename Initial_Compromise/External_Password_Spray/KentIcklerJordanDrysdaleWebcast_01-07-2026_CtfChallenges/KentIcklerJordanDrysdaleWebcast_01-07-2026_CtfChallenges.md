<a name="_sqlwtbik4zk"></a>**Level 1: Reconnaissance & Vulnerability Identification**

Data Snippet (Nmap Scan Output):

PORT STATE SERVICE VERSION\
` `21/tcp open ftp vsftpd 2.3.4\
` `22/tcp open ssh OpenSSH 4.7p1 Debian 8ubuntu1\
` `80/tcp open http Apache httpd 2.2.8 ((Ubuntu) DAV/2)\
` `3306/tcp open mysql MySQL 5.0.51a-3ubuntu5

Based on the service versions identified above, which port represents a service with a well-known "Backdoor Command Execution" vulnerability that often provides immediate root access?

- A) Port 22 (OpenSSH)
- B) Port 80 (Apache)
- C) Port 3306 (MySQL)
- D) Port 21 (vsftpd)

**Answer:** D) Port 21 (vsftpd)

<a name="_la70gc4ty0gf"></a>**Level 2: Web Log Analysis**

Data Snippet (Apache Access Log):

10\.10.14.5 - - [01/Jan/2024:10:05:22] "GET /login.php?user=admin'# HTTP/1.1" 200 4520\
` `10.10.14.5 - - [01/Jan/2024:10:05:25] "GET /search.php?q=<script>alert(1)</script> HTTP/1.1" 200 1205\
` `10.10.14.5 - - [01/Jan/2024:10:05:28] "GET /../../../../etc/passwd HTTP/1.1" 200 1850

Reviewing the three requests in the log above, which specific attack vector was successfully executed in the third request (timestamp 10:05:28), indicated by the response size?

- A) SQL Injection (SQLi)
- B) Local File Inclusion 
- C) Reflected XSS
- D) Brute Force Attack

**Answer:** B) Local File Inclusion





![ref1]**Level 3: Traffic Analysis**

Data Snippet (Wireshark Capture - HTTP Header):

` `GET /admin/dashboard HTTP/1.1\
` `Host: 192.168.1.105\
` `User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)\
` `Cookie: auth\_token=YWRtaW46cGFzc3dvcmQxMjM=;\
` `Accept: text/html,application/xhtml+xml

An attacker intercepts the request above using Burp Suite. Focusing on the Cookie header, what is the immediate risk visible in this data?

- A) The User-Agent suggests an outdated browser vulnerability
- B) The host IP is a private address, indicating an internal leak
- C) The auth\_token is a weak 	Base64 encoding of cleartext credentials
- D) The Accept header allows for XML External Entity (XXE) injection

**Answer:** C) The auth\_token is a weak Base64 encoding of cleartext credentials

![ref1]

<a name="_6z153gq5gpxx"></a>**Level 4: The Hash Challenge**

Scenario: During post-exploitation, you dumped the database of a compromised web application. You found a password hash for the root user stored in the format used by modern Linux systems.

Data Snippet:

$2b$10$hlB.2JLJ2gPMBWmIjglQsOg9bL6bhMj2rZsMuNv1V718u/9bJsjmK

What is the plaintext password?

**Answer:** agoldentouch


[ref1]: Aspose.Words.0537e379-66fe-4181-9ab3-b8d191e53ae1.001.png
