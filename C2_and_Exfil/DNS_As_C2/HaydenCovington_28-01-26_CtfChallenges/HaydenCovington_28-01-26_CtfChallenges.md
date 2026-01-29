<a name="summary"></a>Summary

<a name="setup"></a>Setup

<a name="prerequisites-recommended"></a>Prerequisites (recommended)

*# Install required tools*\
sudo apt update\
sudo apt install -y tshark wireshark

<a name="getting-started"></a>Getting Started

1. Analyze the PCAP file: soc\_ctf.pcap
1. Find the flag!

<a name="challenge-answers"></a>Challenge Answers

<a name="challenge-1-dns-exfiltration"></a>Challenge 1: DNS Exfiltration

**Scenario**: Suspicious DNS queries detected from internal host, data being exfiltrated via DNS subdomains.

**Solution**: Extract DNS queries and decode subdomain patterns

tshark -r files/soc\_ctf.pcap -Y "dns" -T fields -e dns.qry.name **|** grep exfil.attacker.com\
tshark -r files/soc\_ctf.pcap -Y "dns" -T fields -e dns.qry.name **|** grep exfil.attacker.com **|** cut -d. -f1 **|** tr -d '\n'

**Flag**: FLAG{dns\_exfiltrated\_data}

<a name="challenge-2-ftp-file-retrieval"></a>Challenge 2: FTP File Retrieval

**Scenario**: FTP session where file flag.txt was downloaded from internal server.

**Solution**: Recover file from FTP data connection (port 20) and decode hex payload

tshark -r files/soc\_ctf.pcap -Y "tcp.port==20" -T fields -e tcp.payload\
tshark -r files/soc\_ctf.pcap -Y "tcp.port==20" -T fields -e tcp.payload **|** tr -d '\n' **|** xxd -r -p

**Flag**: FLAG{ftp\_transmission\_complete}

<a name="challenge-3-http-credential-harvesting"></a>Challenge 3: HTTP Credential Harvesting

**Scenario**: Suspicious HTTP traffic with credentials being submitted to web form.

**Solution**: Extract password from HTTP POST request body

tshark -r files/soc\_ctf.pcap -Y "http.request.method == POST" -T fields -e http.file\_data **|** xxd -r -p

**Flag**: FLAG{http\_post\_password}
