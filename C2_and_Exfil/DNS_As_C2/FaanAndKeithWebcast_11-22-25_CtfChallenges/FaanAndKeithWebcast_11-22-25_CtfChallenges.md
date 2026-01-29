||||
| :- | :-: | -: |

For solving the following questions, participants are advised to use [RITA](https://www.activecountermeasures.com/free-tools/rita/)

1\. What is the compromised host’s IPv6 address?

Answer: 2604:a880:4:1d0::6a26:c000

How to solve: Look at RITA, the compromised machine is the one initiating outbound traffic to multiple suspicious IPv6 hosts.

2\. List the suspicious IPv6 addresses used by the attacker (same subnet).

Answer:

2604:a880:800:14::ae20:d001

2604:a880:800:14::ae20:d002

2604:a880:800:14::ae20:d003

2604:a880:800:14::ae20:d004

2604:a880:800:14::ae20:d005

How to solve: Simply inspect the connection list. They differ only in the last hex character.

3\. Using an IP lookup service, identify which provider owns the suspicious IPv6 block.

Answer: DigitalOcean, LLC (New York / US)

How to solve (step-by-step) Pick an address, e.g.: 2604:a880:800:14::ae20:d001 Run an ASN lookup: 

whois 2604:a880:800:14::ae20:d001 | grep -Ei "OrgName|netname|descr|owner"

or use the website (as intended):

go to whatismyipaddress.com/ip/2604:a880:800:14::ae20:d001

You’ll see ownership → DigitalOcean.

4\. Determine the smallest IPv6 subnet they can all belong to.

Answer: 2604:a880:800:14::ae20:d000/124

How to solve: Expand the addresses, compare bits to find the common prefix, count identical bits for the prefix length, and set remaining bits to 0 for the network address.

5\. Explain why IPv6 address aliasing increases stealth for C2 traffic (2+ reasons).

Answer:

Difficult to blacklist:\
The attacker rotates the last block of the IPv6 address, generating thousands of valid addresses in the same /64, making static IP-based blocking ineffective.

Flows appear short-lived:\
Each connection goes to a slightly different address, so no single destination accrues enough traffic volume to appear suspicious.

Many security tools ignore IPv6:\
Some SOCs still rely heavily on IPv4-based detections, meaning unusual IPv6 usage often flies under the radar.

Harder for analysts to correlate:\
Traditional grouping (“same IP = same host”) breaks because dozens of addresses map to the same machine.
||||
| :- | :-: | -: |
