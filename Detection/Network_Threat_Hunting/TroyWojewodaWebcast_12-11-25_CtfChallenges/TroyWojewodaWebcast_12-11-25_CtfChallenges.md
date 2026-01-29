Unify Network and Endpoint Security

Hands-On Beaconing, DGA & Exfiltration Analysis

# **CHALLENGES**

## **🔴 Challenge 1 --- Identify the Beaconing Endpoint**

Using:\
• zeek_conn.csv\
• zeek_dns.csv

Your task:\
• Identify which internal IP is beaconing\
• Confirm the beaconing interval\
• Determine whether this is DNS-based or TCP-based C2

## **🔴 Challenge 2 --- Link Beaconing to a User and Process**

Using:\
• zeek_conn.csv\
• mde_devices.csv

Your task:\
• Identify the device name generating the beacon\
• Identify the logged-in user\
• Identify the process responsible

## **🔴 Challenge 3 --- Detect Data Exfiltration**

Using only:\
• zeek_conn.csv

Your task:\
• Identify which internal IP performed exfiltration\
• Identify the destination IP\
• Determine whether this was likely upload or download\
• Estimate the exfiltrated data volume

## **🔴 Challenge 4 --- Detect Algorithmic Domain Generation (DGA)**

Using:\
• zeek_dns.csv

Your task:\
• Identify which domains appear algorithmically generated\
• Identify the infected host\
• Prove that the domain rotation happens at a fixed interval

# **FULL ANSWER KEY (With Commands & Reasoning)**

## **✅ Solution 1 --- Beaconing Endpoint**

Result:\
10.0.0.15 → 45.77.22.10 at 10:01 and 10:02

Beacon interval: 60 seconds\
Beacon type: TCP-based C2\
Infected IP: 10.0.0.15

## **✅ Solution 2 --- User & Process Attribution**

Result:\
Device: DEV-01\
IP: 10.0.0.15\
User: Bartosz\
Process: powershell.exe

Beacon caused by: powershell.exe\
Logged-on user: Bartosz\
Device: DEV-01

This indicates fileless malware or LOLBins abuse.

## **✅ Solution 3 --- Exfiltration Detection**

From zeek_conn.csv:\
10.0.0.23 → 91.222.13.5\
orig_bytes: 1500\
resp_bytes: 98000

Exfiltrating host: 10.0.0.23\
Destination: 91.222.13.5\
Direction: Upload\
Volume: \~98 KB

## **✅ Solution 4 --- DGA Detection**

From zeek_dns.csv:\
a92dk3q.biz\
b93fk1q.biz\
c01lk9q.biz

Random character structure\
Timestamps exactly 60s apart\
Same querying host: 10.0.0.15

This is algorithmic domain rotation for C2 resilience.
