# []{#anchor}CTF Challenge: Quiet Resolution

Backdoors & Breaches Card: C2 & Exfil \-\-- DNS as C2\
\
\

## []{#anchor-1}Challenge Description

A Zeek DNS sensor was deployed to improve network visibility.\
\
No alerts fired, but analysts suspect one internal system may be
communicating with external infrastructure in a way consistent with
command-and-control.\
\
You are provided with a single Zeek dns.log\
\
You are tasked to:\
- Identify the internal host exhibiting suspicious behavior\
- Identify the domain involved\
- Determine the approximate beacon interval\
\
The domain in question is used legitimately elsewhere in the
environment, so reputation-based detection will not be sufficient\
\
Focus on behavior, not signatures

## []{#anchor-2}Flag Format

MetaCTF{\<host_ip\>\|\<domain\>\|\<interval\>}

## []{#anchor-3}Flag Validation Rules

The interval value must be a whole number between and including 20 and
30\
\
Example valid flags:\
CTF{10.47.2.100\|ise.wrccdc.org\|27}\
CTF{10.47.2.100\|ise.wrccdc.org\|26}\
CTF{10.47.2.100\|ise.wrccdc.org\|30}

------------------------------------------------------------------------------------------------------------------------

# []{#anchor-4}**Write-Up**

# []{#anchor-5}Quiet Resolution - Quick Write-Up (Commands + Outputs)

This is a command-by-command walkthrough using the same commands run
during the analysis. Each step includes the exact output observed and a
short explanation of what the command is doing and what the output
shows.

## []{#anchor-6}Step 0 - Confirm the file exists and has reasonable size

Command:\
*ls -lh dns.log*

Output:\
\
\

-rw-r\--r\--. 1 panterasbook29 panterasbook29 9.7M Apr 12 2024 dns.log

This verifies we're working with the expected file and gives a quick
sanity check on size. A \~9.7MB dns.log is large enough to contain
meaningful behavior but still manageable for CLI analysis.

## []{#anchor-7}Step 1 - Confirm the Zeek field layout

Command:\
*grep \'\^#fields\' dns.log*

Output:\
\
\

#fields ts uid id.orig_h id.orig_p id.resp_h id.resp_p proto trans_id
rtt query qclass qclass_name qtype qtype_name rcode rcode_name AA TC RD
RA Z answers TTLs rejected

Zeek logs are column-based. This line is the authoritative schema. We
specifically rely on: \$1 (ts), \$3 (id.orig_h), \$10 (query), \$14
(qtype_name), and \$16 (rcode_name).

## []{#anchor-8}Step 1b - Spot-check the header + a few rows

Command:\
*head -n 30 dns.log*

Output:\
\
\

#separator \\x09\
#set_separator ,\
#empty_field (empty)\
#unset_field -\
#path dns\
#open 2024-04-12-19-29-12\
#fields ts uid id.orig_h id.orig_p id.resp_h id.resp_p proto trans_id
rtt query qclass qclass_name qtype qtype_name rcode rcode_name AA TC RD
RA Z answers TTLs rejected\
#types time string addr port addr port enum count interval string count
string count string count string bool bool bool bool count
vector\[string\] vector\[interval\] bool\
1521911720.865716 CqKst53mF3det3eDV9 10.47.1.100 41772 10.0.0.100 53 udp
36329 0.000870 ise.wrccdc.org 1 C_INTERNET 1 A 0 NOERROR F F TT 0
ise.wrccdc.cpp.edu,134.71.3.16 2230.000000,41830.000000 F\
1521911720.865717 CqKst53mF3det3eDV9 10.47.1.100 41772 10.0.0.100 53 udp
36329 0.000871 ise.wrccdc.org 1 C_INTERNET 1 A 0 NOERROR F F TT 0
ise.wrccdc.cpp.edu,134.71.3.16 2230.000000,41830.000000 F\
1521911720.865911 C8bqc84K9TqNqzE9Yd 10.47.1.100 53995 10.0.0.100 53 udp
35149 0.000827 ise.wrccdc.org 1 C_INTERNET 1 A 0 NOERROR F F TT 0
ise.wrccdc.cpp.edu,134.71.3.16 2230.000000,41830.000000 F\
\...

This confirms the separator is a tab (\\x09) and shows real records
beneath the header. It also helps validate that field positions match
what we'll reference in awk.

## []{#anchor-9}Step 2 - Find the top DNS talkers (source IPs)

Command:\
*grep -v \'\^#\' dns.log \| awk -F\'\\t\' \'{print \$3}\' \| sort \|
uniq -c \| sort -nr \| head -n 15*

Output:\
\
\

9346 10.47.2.100\
4284 10.47.6.154\
3422 10.47.1.208\
2376 10.47.5.155\
2322 10.47.1.100\
2270 172.31.255.5\
2178 10.47.5.100\
1826 10.47.8.155\
1713 10.47.3.142\
1654 10.47.1.153\
1610 10.47.2.155\
1596 10.47.6.10\
1572 10.47.4.100\
1532 10.47.1.10\
1492 10.47.7.10

This strips headers, prints the source IP column (id.orig_h), counts
occurrences per IP, then sorts descending. The output shows 10.47.2.100
generated the most DNS log entries in this dataset.

## []{#anchor-10}Step 3 - For 10.47.2.100, list most-queried domains

Command:\
*IP=\"10.47.2.100\"\
grep -v \'\^#\' dns.log \| awk -F\'\\t\' -v ip=\"\$IP\" \'\$3==ip {print
\$10}\' \| sort \| uniq -c \| sort -nr \| head -n 40*

Output:\
\
\

7290 ise.wrccdc.org\
120 registry.npmjs.org\
104 docs.google.com\
96 ws.gitter.im\
84 clients4.google.com\
74 notifications.google.com\
74 collector-pxpmp8ilui.perimeterx.net\
70 ssl.gstatic.com\
68 www.google.com\
68 play.google.com\
64 ogs.google.com\
64 github.com\
54 0.docs.google.com\
50 avatars2.githubusercontent.com\
50 avatars1.githubusercontent.com\
48 clients6.google.com\
42 avatars0.githubusercontent.com\
42 assets-cdn.github.com\
36 www.gstatic.com\
36 apis.google.com\
36 adservice.google.com\
34 cello.client-channel.google.com\
34 avatars3.githubusercontent.com\
32 www.google-analytics.com\
32 safebrowsing.google.com\
32 safebrowsing-cache.google.com\
26 \_ipp.\_tcp.local\
24 fonts.gstatic.com\
24 drive.google.com\
24 collector.githubapp.com\
20 id.google.com\
20 api.github.com\
20 accounts.google.com\
18 www2.deloitte.com\
18 fonts.googleapis.com\
14 lh3.googleusercontent.com\
12 www.googletagmanager.com\
12 gitter.im\
12 cdn03.gitter.im\
12 cdn02.gitter.im

This filters dns.log down to one host and counts how often each domain
appears. The key detail in the output is that ise.wrccdc.org dominates
the host's DNS activity (7290 queries).

## []{#anchor-11}Step 3b - Baseline check with another noisy host (10.47.6.154)

Command:\
*IP=\"10.47.6.154\"\
grep -v \'\^#\' dns.log \| awk -F\'\\t\' -v ip=\"\$IP\" \'\$3==ip {print
\$10}\' \| sort \| uniq -c \| sort -nr \| head -n 40*

Output:\
\
\

3464 ise.wrccdc.org\
100 arena1.wrccdc.cpp.edu\
54 ssl.gstatic.com\
28 www.digitalocean.com\
28 notifications.google.com\
24 ogs.google.com\
24 cello.client-channel.google.com\
22 play.google.com\
22 pastebin.com\
20 www.linux.com\
20 www.google.com\
20 tiles.services.mozilla.com\
20 productsearch.ubuntu.com\
18 apis.google.com\
18 adservice.google.com\
16 unix.stackexchange.com\
16 superuser.com\
16 stackoverflow.com\
16 github.com\
16 askubuntu.com\
12 www.gstatic.com\
12 www.cyberciti.biz\
12 \_ldap.\_tcp.dc.\_msdcs.factory.oompa.loompa\
12 id.google.com\
10 www.nytimes.com\
10 lh3.googleusercontent.com\
10 clients4.google.com\
8 www.theverge.com\
8 www.mozilla.org\
8 www.linuxquestions.org\
8 use.typekit.net\
8 getpocket.com\
8 drive.google.com\
6 www.googletagservices.com\
6 staticxx.facebook.com\
6 lqo-thequestionsnetw.netdna-ssl.com\
6 img-getpocket.cdn.mozilla.net\
6 go.digitalocean.com\
6 fonts.googleapis.com\
6 cdn.polyfill.io

Same breakdown for a different host. This provides context: 10.47.6.154
still queries ise.wrccdc.org often, but also shows a more varied set of
domains (including arena1.wrccdc.cpp.edu).

## []{#anchor-12}Step 4 - Pull timestamps for the suspect host+domain

Command:\
*IP=\"10.47.2.100\"\
DOM=\"ise.wrccdc.org\"\
grep -v \'\^#\' dns.log \| awk -F\'\\t\' -v ip=\"\$IP\" -v dom=\"\$DOM\"
\'\$3==ip && \$10==dom {print \$1}\' \| head -n 30*

Output:\
\
\

1521911747.886585\
1521911747.886638\
1521911747.886642\
1521911747.886644\
1521911747.992155\
1521911747.992157\
1521911747.992159\
1521911747.992161\
1521911747.993289\
1521911747.993293\
1521911747.997453\
1521911747.997546\
1521911747.997550\
1521911747.997552\
1521911748.000462\
1521911748.000464\
1521911748.000466\
1521911748.000502\
1521911748.003348\
1521911748.003349\
1521911748.003351\
1521911748.003353\
1521911748.007595\
1521911748.007599\
1521911748.007600\
1521911748.007602\
1521911748.010157\
1521911748.010159\
1521911748.010162\
1521911748.010164

This prints the timestamp column (ts) for matching records. The output
shows bursts of requests close together. At this stage we don't assume
beaconing; we just confirm we can isolate events for interval analysis.

## []{#anchor-13}Step 4b - Compute raw time deltas (unsorted, all record types)

Command:\
*IP=\"10.47.2.100\"\
DOM=\"ise.wrccdc.org\"\
grep -v \'\^#\' dns.log \| awk -F\'\\t\' -v ip=\"\$IP\" -v dom=\"\$DOM\"
\'\
\$3==ip && \$10==dom {\
t=\$1\
if (prev != \"\") {\
printf \"%.6f\\n\", t-prev\
}\
prev=t\
}\' \| head -n 50*

Output:\
\
\

0.000053\
0.000004\
0.000002\
0.105511\
0.000002\
0.000002\
0.000002\
0.001128\
0.000004\
0.004160\
0.000093\
0.000004\
0.000002\
0.002910\
0.000002\
0.000002\
0.000036\
0.002846\
0.000001\
0.000002\
0.000002\
0.004242\
0.000004\
0.000001\
0.000002\
0.002555\
0.000002\
0.000003\
0.000002\
0.002555\
0.000004\
0.000002\
0.000002\
0.003072\
0.000002\
0.000002\
0.000001\
0.003634\
0.000003\
0.000001\
0.000002\
0.008010\
0.000002\
0.563523\
0.000046\
-0.000003\
0.000005\
0.070867\
0.000002\
0.000005

This is a quick first pass to see what the gaps look like. The output is
dominated by tiny gaps (milliseconds and below), plus occasional larger
gaps. A small negative delta can occur due to ordering/merge effects,
which is why we normalize next.

## []{#anchor-14}Step 5 - Normalize by filtering to A records and sorting, then compute deltas

Command:\
*IP=\"10.47.2.100\"\
DOM=\"ise.wrccdc.org\"\
\
grep -v \'\^#\' dns.log \\\
\| awk -F\'\\t\' -v ip=\"\$IP\" -v dom=\"\$DOM\" \'\$3==ip && \$10==dom
&& \$14==\"A\" {print \$1}\' \\\
\| sort -n \\\
\| awk \'NR==1{prev=\$1; next} {d=\$1-prev; if(d\>=0) print d;
prev=\$1}\' \\\
\| head -n 80*

Output:\
\
\

5.29289e-05\
0.105517\
1.90735e-06\
0.00113201\
4.05312e-06\
0.00415993\
9.29832e-05\
0.0029161\
1.90735e-06\
0.00288415\
9.53674e-07\
0.004246\
4.05312e-06\
0.00255799\
1.90735e-06\
0.0025599\
4.05312e-06\
0.00307608\
1.90735e-06\
0.00363708\
2.86102e-06\
0.00801301\
2.14577e-06\
0.563523\
4.29153e-05\
0.0708721\
1.90735e-06\
1.00678\
9.53674e-07\
0.071696\
9.53674e-07\
0.00339508\
3.8147e-06\
0.0135672\
1.90735e-06\
0.00334692\
2.14577e-06\
0.00290084\
4.05312e-06\
0.00239801\
3.09944e-06\
0.00362802\
1.90735e-06\
0.00275898\
5.00679e-06\
0.00823402\
4.05312e-06\
0.115117\
2.14577e-06\
0.00424194\
3.09944e-06\
0.00283885\
3.09944e-06\
0.00295305\
3.8147e-06\
0.00282502\
4.05312e-06\
0.00281811\
2.86102e-06\
0.00245309\
1.90735e-06\
0.00459194\
2.14577e-06\
0.00316906\
3.8147e-06\
0.00470805\
2.14577e-06\
0.163989\
2.14577e-06\
0.121852\
4.05312e-06\
0.00865197\
1.90735e-06\
0.00392008\
1.90735e-06\
0.0287662\
2.86102e-06\
0.0814371\
2.86102e-06\
0.00470901

Filtering to qtype_name==A removes extra noise from mixed record types
(A vs AAAA). Sorting ensures deltas are computed in time order. The
output still includes micro-gaps, but now the larger gaps (e.g.,
\~1.00678 and \~0.563523) are trustworthy.

## []{#anchor-15}Step 5b - Show only gaps \>= 1 second (macro timing)

Command:\
*IP=\"10.47.2.100\"\
DOM=\"ise.wrccdc.org\"\
\
grep -v \'\^#\' dns.log \\\
\| awk -F\'\\t\' -v ip=\"\$IP\" -v dom=\"\$DOM\" \'\$3==ip && \$10==dom
&& \$14==\"A\" {print \$1}\' \\\
\| sort -n \\\
\| awk \'NR==1{prev=\$1; next} {d=\$1-prev; if(d\>=1) print d;
prev=\$1}\' \\\
\| head -n 80*

Output:\
\
\

1.00678\
28.4156\
1.29892\
29.0177\
1.12642\
1.33006\
28.0721\
28.5102\
1.17912\
26.7538\
2.42461\
26.8915\
2.57011\
26.5676\
2.53961\
26.3692\
2.34179\
16.8417\
11.5515\
1.09195\
1.9906\
3.01954\
2.26701\
9.47339\
19.4038\
5.00445\
4.57257\
1.2342\
18.6988\
8.20996\
18.8354\
1.27986\
1.03815\
7.33345\
19.4033\
6.90276\
19.9683\
1.87336\
3.66863\
1.10265\
1.50809\
25.8454\
1.71096\
2.15819\
24.7239\
1.5366\
1.00502\
1.75593\
1.74987\
23.8914\
1.51191\
1.42815\
81.5321\
29.6185\
1.40608\
28.1797\
1.61321\
26.9249\
1.74815\
1.12708\
26.3517\
2.36895\
1.11872\
1.24436\
29.2384\
1.32475\
28.3411\
1.38622\
1.7868\
1.14433\
26.6124\
2.04155\
25.7342\
2.51888\
25.3928\
2.16453\
26.643\
2.51637\
1.69039\
5.00392

This filters the deltas down to the gaps that matter for beacon-style
behavior. The output shows repeated gaps in the high-20s (26--29
seconds) alongside other occasional gaps.

## []{#anchor-16}Step 7 - Check how many hosts query the same domain

Command:\
*DOM=\"ise.wrccdc.org\"\
grep -v \'\^#\' dns.log \\\
\| awk -F\'\\t\' -v dom=\"\$DOM\" \'\$10==dom {print \$3}\' \\\
\| sort \| uniq -c \| sort -nr \| head -n 15*

Output:\
\
\

7290 10.47.2.100\
3464 10.47.6.154\
2002 10.47.5.100\
1794 10.47.1.100\
1464 10.47.4.100\
1336 10.47.8.155\
1116 10.47.5.155\
1020 10.47.1.153\
780 10.47.2.153\
638 10.47.4.155\
494 10.47.2.155\
282 10.47.6.173\
266 10.47.1.151\
202 10.47.5.156\
12 10.47.3.156

This counts how many queries to the domain come from each source IP. The
output shows the domain is used by multiple internal hosts, which means
reputation alone won't help. The standout remains 10.47.2.100 based on
both volume and timing behavior.

## []{#anchor-17}Step 8 - Interval fingerprint (rounded, 20--40 second window)

Command:\
*IP=\"10.47.2.100\"\
DOM=\"ise.wrccdc.org\"\
\
grep -v \'\^#\' dns.log \\\
\| awk -F\'\\t\' -v ip=\"\$IP\" -v dom=\"\$DOM\" \'\$3==ip && \$10==dom
&& \$14==\"A\" {print \$1}\' \\\
\| sort -n \\\
\| awk \'NR==1{prev=\$1; next} {d=\$1-prev; if(d\>=20 && d\<=40) print
int(d+0.5); prev=\$1}\' \\\
\| sort -n \\\
\| uniq -c \\\
\| sort -nr*

Output:\
\
\

6 27\
4 28\
4 26\
3 29\
3 25\
3 21\
2 24\
1 30\
1 23\
1 22

This produces a simple histogram of rounded deltas between 20 and 40
seconds. The output shows a cluster around 27--29 seconds (with 27 most
frequent).

## []{#anchor-18}Result Used for the Challenge

Host: 10.47.2.100\
Domain: ise.wrccdc.org\
Interval: accept any integer 20--30 (inclusive) per challenge rules.
