Description:

Your organization suspects unauthorized access to its Microsoft 365 tenant.

No malware.\
` `No phishing alerts.\
` `No user complaints.

However a sensitive email has leaked.

You are provided with several exports collected during an incident response investigation, including: exchange online mail flow rules, mailbox inbox rules, administrative audit events, role assignments, message trace data

Find the flag in the subject of the leaked email.

Difficulty: 225 points

Flag: MetaCTF{un4uthor1z3d\_4cc3ss}

Solution:

First, inspect transport\_rules.json and find the only enforced rule that silently BCCs forwarded mail to an external domain (<archive@mx.shadow-mail.net>).

\
` `Next, confirm from audit\_log\_excerpt.json and user\_roles.csv that this rule and a suspicious inbox rule were created by an Exchange Administrator, indicating unauthorized cloud configuration changes.

\
` `Then, use the transport rule’s conditions to filter message\_trace.csv for messages where AutoForwarded=true and the BCC target matches the attacker’s external address.\
` `Finally, take the matching message and Base64-decode its Subject to recover the flag.
