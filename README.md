# MY-FOOTPRINTING-AND-SCANNING-TRIALS
 


Pentester Name	Ilemobayo John Oluwaseun
(Cybersecurity Intern)
Program / Batch	B084 - Networkwalks Cybersecurity Internship
Date of testing	10 - 11 November 2026

Modules covered	W2-PM1 (Multiple Kali Tools) - Completed
W2-PM5 (Zenmap Scanning) - Completed


Client / Target	1.	Networkwalks (networkwalks.com) - written permission secured
2.	My own local LAN network (192.168.10.x)
3.	Practice targets not named in the Letter: scanme.nmap.org (Nmap Project public test host) and microsoft.com (passive theHarvester lookup only). See the scope note in Section 1.

Permission secured from client?	Yes
Targets 1 and 2: Letter of Authorization Ref. NW-LOA-B082-017, valid 08 - 13 November 2026 (see Section 8). Target 3 is covered by the scope note in Section 1.

Phases covered	Phase 1: Reconnaissance & Footprinting –
 Completed Phase 2: Scanning & Network Discovery - Completed Phase 3-5: Outside the scope of the authorization

1.	Liability Disclaimer
I have performed these activities only on systems and devices where I had secured written permission, or on devices and networks that I own myself. All of this material is for education and research purposes only. Do not use anything from this report to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.
Authorization: Networkwalks issued a Letter of Authorization (Ref. NW-LOA-B082-017) covering reconnaissance and scanning of networkwalks.com and its DNS records, plus my own LAN. The footprinting of networkwalks.com (10 November 2026) and the scans of my own LAN (10 - 11 November 2026) were carried out under that letter, inside its validity window of 08 - 13 November 2026, using only the permitted Phase 1 and Phase 2 activities.
Scope note: Two practice exercises used targets that are not named in the Letter of Authorization: (1) theHarvester lookups for microsoft.com, which query public third-party data sources and do not scan the target's own systems, and (2) one Nmap scan of scanme.nmap.org, a test host that the Nmap Project publishes for scan practice. They are listed separately in this
 
report for transparency. In every activity, no exploitation, brute-force or denial-of-service activity was attempted.

2.	Introduction
This report covers two practical modules. W2-PM1 covers footprinting the networkwalks.com domain with multiple Kali Linux tools, plus a practice run of theHarvester. W2-PM5 covers scanning my own local network with Zenmap, plus one practice scan of a public test host.
Together they show how an attacker moves from gathering public information to mapping live hosts, open ports and services. This is the Week 2 part of my ongoing internship program at Networkwalks.
The footprinting commands were run in a Kali Linux virtual machine in VirtualBox. The scans were run with Zenmap on my Windows 11 PC. Every step below includes the exact command used, the result I observed, a screenshot as evidence, and a short note on why the finding matters from an attacker's point of view.

3.	Tools Used
The table below lists each tool used in this report and its purpose.

Tool	Purpose
Kali Linux / Windows 11	Operating systems used: Kali Linux (VirtualBox VM) for footprinting and a Windows 11 PC for Zenmap scanning
WHOIS	Find domain registration details (registrar, dates, name servers)
whatweb	Fingerprint web technologies (server, CMS, plugins, IP)
nslookup	Resolve the domain name to its IP address using DNS
curl -I	Read the HTTP response headers of the website
wafw00f	Detect whether a Web Application Firewall protects the site
dnsrecon	Enumerate DNS records (SOA, NS, MX, A, TXT, SRV)
theHarvester	Collect emails, hosts and subdomains from public sources (OSINT)
Zenmap (Nmap GUI)	Discover hosts, open ports, services and OS details, and draw the network topology

4.	Activities Performed
4.1	Footprinting & Reconnaissance
I performed reconnaissance against the networkwalks.com domain using six Kali Linux tools: WHOIS, WhatWeb, Nslookup, Curl, Wafw00f and DNSRecon. Each tool was used to collect a different type of information about the target.
4.1.1	WHOIS - Domain registration information
Command:

 
Result:
•	Registrar: GoDaddy.com, LLC (IANA ID 146); abuse contact abuse@godaddy.com.
•	Domain created 6 November 2019, last updated 12 November 2025, registration expires 6
November 2027.
•	Registrant and technical contact details are hidden behind a privacy service (Domains By Proxy, LLC, Tempe, Arizona).
•	Domain status locks are set: clientTransferProhibited, clientUpdateProhibited, clientRenewProhibited and clientDeleteProhibited.
•	Name servers: NS6135.HOSTGATOR.COM and NS6136.HOSTGATOR.COM, plus NS29.DOMAINCONTROL.COM and NS30.DOMAINCONTROL.COM in the registrar
record.
•	DNSSEC: unsigned.

Figure 1: WHOIS output for networkwalks.com (registry record)
 

 
Figure 2: WHOIS output continued (registrar record begins)

Figure 3: WHOIS output continued (registrant, technical contact and name servers)
 

 
Figure 4: WHOIS output continued (end of record)
Why it matters: WHOIS tells an attacker who the registrar is, how old the domain is, and where DNS is hosted (HostGator and GoDaddy name servers), which helps them plan further research on the hosting provider. The privacy service and the domain locks are good practice because they hide personal details and make unauthorised transfers harder.
4.1.2	WhatWeb - Web technology fingerprinting
Command:

Result:
•	The plain HTTP request returned 301 Moved Permanently and redirected to https://networkwalks.com/, which then returned 200 OK.
•	Web server: Apache. IP address: 192.232.216.135. Country: United States.
•	CMS: WordPress 7.1.4 with the WordPress Download Manager plugin 3.3.76 (from the MetaGenerator tag). Also detected: jQuery 3.7.1, Google Tag Manager and HTML5.
•	Page title: "Networkwalks Academy". A public contact email, info@networkwalks.com, was found on the page.
•	A cookie named   wpdm_client is set with the HttpOnly flag.

Figure 5: WhatWeb output for networkwalks.com
 
Why it matters: Knowing the CMS, plugin and library versions lets an attacker check them against public vulnerability lists and choose what to research next. The exposed email address can also be used for phishing or social engineering.
4.1.3	Nslookup - DNS name resolution
Command:

Result:
•	The domain resolved to 192.232.216.135 as a non-authoritative answer via DNS server 8.8.8.8.
•	The output shows a "communications error to 8.8.8.8#53: timed out" warning before the answer was returned.

Figure 6: Nslookup output for networkwalks.com
Why it matters: The IP address is the starting point for infrastructure mapping. The same address appears in WhatWeb and as the MX host in the DNSRecon results, which shows the web and mail services share a server.
4.1.4	Curl - HTTP response headers
Command:

Result:
•	Response: HTTP/2 200 with server: Apache.
•	A link header exposes the WordPress REST API at /wp-json/ and a JSON route for page 53 (/wp-json/wp/v2/pages/53).
•	Caching headers x-nginx-cache: WordPress and x-endurance-cache-level: 0 reveal hosting and caching details.
•	The   wpdm_client cookie is set with the Secure and HttpOnly flags.
 
•	Referrer-Policy is no-referrer-when-downgrade, and a Permissions-Policy header is present. Headers such as Strict-Transport-Security, Content-Security-Policy, X-Frame-Options and X-Content-Type-Options were not seen in this response.

Figure 7: Curl -I output showing the HTTP response headers
Why it matters: Headers give away the server software and hosting setup, and the exposed
/wp-json/ endpoint tells an attacker the site runs the WordPress REST API, which is a common target for enumeration. Missing hardening headers reduce browser-side protection.
4.1.5	Wafw00f - Web Application Firewall detection
Command:

Result:
•	Wafw00f v2.4.2 reported that https://networkwalks.com is behind the ModSecurity (SpiderLabs) WAF, using 2 requests.

Figure 8: Wafw00f detecting ModSecurity (SpiderLabs)
Issue encountered: In one run, wafw00f failed with "Temporary failure in name resolution" and wrongly reported the site as down (Figure 9). This was a name-resolution problem on my virtual machine, not a problem with the target. A successful run (Figure 8) returned the result above.
 

 
Figure 9: Wafw00f run that failed because of a temporary DNS resolution error
Why it matters: Knowing which WAF protects a site tells an attacker which rule set to research and how to shape later testing. For the defender, having a WAF in place is a positive control.
4.1.6	DNSRecon - DNS record enumeration
Command:

Result:
•	SOA: ns6135.hostgator.com (50.87.144.87).
•	NS: ns6136.hostgator.com (192.232.216.131) and ns6135.hostgator.com (50.87.144.87). Both report BIND version "9.16.23-RH".
•	MX: mail.networkwalks.com (192.232.216.135). A record: networkwalks.com (192.232.216.135).
•	TXT: SPF record "v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all" and a Google site-verification record.
•	No answer for the DNSSEC query, and no SRV records found.

 
Figure 10: DNSRecon output for networkwalks.com
Why it matters: DNS records map out the mail servers, name servers and email-sending policy. The disclosed BIND version on the name servers could help an attacker research known issues, and the soft-fail SPF policy is weaker against email spoofing than a strict one.
 
4.1.7	theHarvester - OSINT collection (practice domain: microsoft.com)
Scope note: microsoft.com is not named in the Letter of Authorization. I used it only as a practice domain for this tool. theHarvester collects data from public third-party sources and does not scan the target's own systems.
Commands:

Result:
•	theHarvester 4.11.1 was used. Its help screen shows options such as -d (domain), -l (result limit), -b (data source), -f (save results) and -s (Shodan), and lists more than 60 supported sources.
•	First run: only the Baidu source was used, with a limit of 1000. It found no IPs, emails, people or hosts.
•	Second run: all sources with a limit of 50. Many sources printed "Missing API key" (for example BuiltWith, Censys, Hunter, Shodan, SecurityTrails, VirusTotal and GitHub), so they were skipped.
•	The sources that need no key (such as Baidu, Certspotter, CRTsh, DuckDuckGo, GitLab, Hackertarget, Mojeek, OTX, RapidDNS, Wayback Archive and Yahoo) were searched. Several returned errors such as "No response from Chaos API", "Expected object or value" and "Could not contact DNS servers".
•	Final result: Hudson Rock found 0 hosts, 0 IPs and 0 emails, and the run ended with no LinkedIn users, IPs, emails, people or hosts found.

Figure 11: theHarvester help screen (version 4.11.1)
 

 
Figure 12: First run: Baidu source only, no results (domain typed as microsft.com)

Figure 13: Second run with all sources: start of output and missing API key messages
 

 
Figure 14: Second run continued: more missing API key messages

Figure 15: Second run continued: sources being searched, with several errors
 

 
Figure 16: End of the second run: no IPs, emails, people or hosts found
Issues encountered: The first command had a typo in the domain (microsft.com), so its empty result is not meaningful. In the second run, most sources were skipped because I had no API keys, and a few could not reach DNS from my virtual machine. No data was returned, which shows the limits of the free, keyless mode.
Why it matters: theHarvester automates the collection of emails, subdomains, hosts and IP addresses from public sources. An attacker uses the results to map an organization's attack surface and to find staff emails for phishing. With API keys the results are usually much richer, so an empty result should never be read as proof that nothing is exposed.
4.2	Network Scanning with Zenmap
For the second activity I used Zenmap (the Nmap GUI) on my Windows 11 PC to scan my own local network. My PC's address is 192.168.10.143, and the first hop of the traceroute to the internet is 192.168.10.1, so my network uses the 192.168.10.x range. Some address details in the screenshots were masked by hand before they were added to this report.
4.2.1	Quick scan of my PC
Profile and command:

Result:
•	Scan started on 10 November 2026 at 21:54. The host was up with a latency of 0.00076 seconds.
•	Of the 100 ports checked, 96 were closed (reset). Four were open: 135/tcp (msrpc), 139/tcp (netbios-ssn), 445/tcp (microsoft-ds) and 554/tcp (rtsp).
•	The scan finished in 1.66 seconds.
 

 
Figure 17: Zenmap Quick scan of 192.168.10.143
Why it matters: A quick scan shows within seconds which common services a host is listening on. Windows networking ports such as 135, 139 and 445 tell an attacker the device is a Windows machine offering file-sharing and RPC services.
4.2.2	Intense scan of my gateway
Profile and command:

Result:
•	Scan started on 10 November 2026 at 21:58. The ARP ping scan found the host up (1 total host).
•	The SYN stealth scan of 1000 ports finished in 0.27 seconds and found these open ports: 80/tcp, 21/tcp, 53/tcp, 2000/tcp and 8291/tcp.
•	The screenshot was taken as the service scan was starting, so service names and versions are not shown.
 

 
Figure 18: Zenmap Intense scan of the gateway (192.168.10.1), partly masked
Why it matters: Open web (80) and FTP (21) ports on a gateway mean it offers services to every device on the LAN. FTP sends data unencrypted, and port 8291 is commonly used for router management, so these services should be limited to the people who need them.
4.2.3	Intense scan of my PC
Profile and command:

Result:
•	Scan started on 11 November 2026 at 07:37 and finished in 29.48 seconds. The host was up with a latency of 0.00015 seconds. 994 ports were closed.
•	Six ports were open: 135/tcp (Microsoft Windows RPC), 139/tcp (Microsoft Windows netbios-ssn), 445/tcp (microsoft-ds), 554/tcp (rtsp), 2869/tcp and 10243/tcp (both Microsoft HTTPAPI httpd 2.0, SSDP/UPnP).
•	The HTTP service header was Microsoft-HTTPAPI/2.0 with the page title "Not Found".
•	OS detection: general purpose device running Microsoft Windows 11 (24H2 - 25H2). Uptime guess: about 5.98 days. Network distance: 0 hops, because this is the PC running the scan.
•	SMB script results: SMB 3.1.1 with message signing enabled and required.
 

 
Figure 19: Intense scan of 192.168.10.143: scan progress and open ports

Figure 20: Intense scan continued: OS detection, uptime and SMB script results
 

 
Figure 21: Ports / Hosts tab showing the six open ports and their versions
Why it matters: An intense scan gives an attacker the operating system, service versions and uptime of a device. File-sharing, RPC and UPnP services can be abused on a network, so they should only be running when needed. Required SMB signing is a good protection.
4.2.4	Network topology
After the intense scan, I opened the Topology tab, turned on the legend and saved the topology graphic with Save Graphic.
•	My PC (192.168.10.143) appears as a yellow circle, which the legend defines as a host with 3 to 6 open ports. This matches the six open ports found.
•	It is connected to "localhost" by a dashed line, which the legend defines as no traceroute information, because the scanned host is the PC itself.

Figure 22: Zenmap Topology tab with the legend open
 



 

Figure 23: Saved topology graphic
Why it matters: A topology map shows how devices are connected and which ones expose the most services, which helps both defenders and attackers decide where to focus.
4.2.5	Practice scan of scanme.nmap.org
Scope note: scanme.nmap.org is not named in the Letter of Authorization. It is a test host that the Nmap Project publishes for scan practice, and I scanned it once.
Profile and command:

Result:
•	Scan started on 11 November 2026 at 07:56 and finished in 39.64 seconds. Host discovery was disabled (-Pn). The name resolves to 2 IPs, and Nmap used 45.33.32.156.
•	993 ports were closed. Open ports: 22/tcp (ssh, OpenSSH 6.6.1p1 Ubuntu), 80/tcp (http, Apache httpd 2.4.7 Ubuntu, page title "Go ahead and ScanMe!") and 9929/tcp (nping-echo). Port 25 (smtp) and ports 135, 139 and 445 were filtered.
•	OS detection gave aggressive guesses of Linux kernels (about 5.0 to 5.14) with no exact match. Uptime guess: about 5.19 days. Network distance: 21 hops.
•	The traceroute started at my gateway (192.168.10.1), passed through several providers, and ended at scanme.nmap.org (45.33.32.156).

Figure 24: Scan of scanme.nmap.org: scan progress
 

 
Figure 25: Scan of scanme.nmap.org: open ports and service versions

Figure 26: Scan of scanme.nmap.org: OS guesses and traceroute

Figure 27: Topology graphic for the scanme.nmap.org traceroute
 
Why it matters: This shows how much a single intense scan can reveal about an internet-facing host: open services with exact software versions, an operating system estimate, and the network path to it. Old versions such as OpenSSH 6.6.1 and Apache 2.4.7 are the kind of detail an attacker would check against known vulnerabilities.

5.	Risk Analysis / Impact
Based on the information collected during the footprinting and network scanning activities, I identified the following potential risks.

#	Risk / Finding	Evidence / Observation	Potential Impact	Risk Level


1	Web technology and version information exposed	WhatWeb identified WordPress 7.1.4, WordPress Download Manager 3.3.76, jQuery
3.7.1 and Apache	Exposed technology and version details can be matched against public advisories to decide what needs further security review	

●	Medium


2	
Server IP address identifiable	nslookup and WhatWeb both returned 192.232.216.135; the MX
record points to the same IP	

Reveals the network location of the web and mail services	

●	Low


3	
HTTP technical information exposed	curl -I showed Server: Apache, x-nginx-cache and x-endurance-cache-level headers, and a link header exposing /wp-json/	Assists technology fingerprinting and enumeration of the WordPress REST API	

●	Low



4	
Common security response headers not observed	HSTS, CSP, X-Frame-
Options and X-Content-Type-Options did not appear in the curl -I response; Referrer-Policy is no-referrer-when-downgrade	Fewer browser-side protections against clickjacking, content sniffing and protocol downgrade (to be confirmed with a full header check)	


●	Low

5	WAF technology identifiable	wafw00f identified ModSecurity (SpiderLabs)	Tells an attacker which rule set to research. The WAF itself is a positive control	
●	Low


6	
DNS server software version disclosed	dnsrecon returned BIND version "9.16.23-RH" for both name servers (192.232.216.131 and
50.87.144.87)	
Version banners help an attacker look for known issues in that release	

●	Medium


7	
Email and DNS hardening observations	SPF record ends in ~all (soft fail); WHOIS shows DNSSEC unsigned and dnsrecon got no DNSSEC answer	
Weaker protection against email spoofing and DNS tampering	

●	Low

8	Public contact email address exposed	WhatWeb found info@networkwalks.com on the home page	May attract phishing and spam or be used in social engineering	
●	Low
 

#	Risk / Finding	Evidence / Observation	Potential Impact	Risk Level



9	
Name server records differ between registry and registrar	Registry record lists NS6135/NS6136.HOSTG
ATOR.COM; registrar record also lists NS29/NS30.DOMAINCON TROL.COM	

Inconsistent records can indicate leftover DNS configuration worth reviewing	


●	Low



10	
Windows file-sharing and RPC ports open on the LAN	Intense scan of my PC found 135 (msrpc), 139
(netbios-ssn) and 445 (microsoft-ds) open. SMB signing is enabled and required	
SMB, NetBIOS and RPC are common targets for lateral movement inside a network. Required SMB signing reduces the risk	


●	Medium


11	
Extra services exposed by the PC	Ports 554 (rtsp), 2869 and
10243 (Microsoft HTTPAPI
httpd 2.0, SSDP/UPnP) were open	
Services that are not needed widen the attack surface and reveal OS and service details	
●	Low


12	
Gateway services reachable from the LAN	
Scan of 192.168.10.1
found 80 (HTTP), 21
(FTP), 53 (DNS), 2000
and 8291 open	FTP is unencrypted, and management-type ports on a gateway are valuable targets. They should be limited to admin use	

●	Medium
Risk level key: ● Critical  ● Medium  ● Low
The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities. The practical exercises involved information gathering, host discovery and port scanning. No exploitation or vulnerability validation was performed. The presence of information such as a software version, IP address, DNS record or open port does not by itself mean that the system is vulnerable. Further authorized security testing would be required to confirm any actual vulnerability.

Positive observations
•	Domain lock statuses are enabled and registrant details are protected by a privacy service.
•	A ModSecurity WAF is protecting the website.
•	HTTP requests are redirected to HTTPS (301 redirect seen in WhatWeb).
•	The session cookie is set with the Secure and HttpOnly flags.
•	No SRV records were exposed in DNS.
•	SMB message signing is enabled and required on my PC.
•	Only six TCP ports were open on my PC, and the other 994 scanned ports were closed.

6.	Recommendations
Based on the observations from these activities, I recommend the following security improvements:
•	Reduce exposed version information: Hide the WordPress generator tag and remove version strings from public pages, plugins and scripts where possible.
•	Keep software updated: Keep WordPress, WordPress Download Manager and other plugins updated and reviewed against current security advisories.
 
•	Harden HTTP headers: Add Strict-Transport-Security, Content-Security-Policy, X-Frame-Options and X-Content-Type-Options, and use a stricter Referrer-Policy such as strict-origin-when-cross-origin. Remove unnecessary technical headers where possible.
•	Review the WordPress REST API: Restrict or disable public REST API routes that are not needed, especially any that reveal user or content details.
•	Hide DNS software versions: Ask the DNS hosting provider to hide the BIND version banner on the name servers.
•	Strengthen email and DNS protections: Review the SPF policy (consider moving from
~all to -all once all legitimate senders are listed), confirm DKIM and DMARC are published, and consider enabling DNSSEC.
•	Review name server records: Check that the registry and registrar name server records match, and remove any that are no longer used.
•	Properly configure and monitor the WAF: Keep ModSecurity enabled and tuned, since it already blocks naive attacks.
•	Protect public contact details: Use a contact form or mail filtering instead of publishing a plain email address, and watch for phishing aimed at it.
•	Maintain domain protections: Keep the domain locks and WHOIS privacy enabled and renew the registration before 6 November 2027.
•	Reduce services on the Windows PC: Turn off services that are not needed (such as UPnP/SSDP and media sharing), keep SMB signing required, and use the firewall so file-sharing ports are not open on untrusted networks.
•	Secure the gateway: Disable FTP and any management service that is not needed, limit admin access to specific trusted devices, change default credentials and keep the router firmware updated.
•	Review OSINT exposure: Organizations should periodically check which emails, subdomains and hosts can be found through search engines and public data sources, since tools like theHarvester collect them automatically.
•	Perform security testing with authorization: Reconnaissance and scanning should only be performed against systems and networks where appropriate written authorization has been provided.

7.	Conclusion
During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, reconnaissance and network scanning.
In the footprinting activity, I used six Kali Linux tools against the target domain, plus theHarvester on a practice domain. I learned how WHOIS can provide domain and name server information, WhatWeb can identify web technologies, Nslookup can resolve domain names, Curl can inspect HTTP headers, Wafw00f can identify a WAF, DNSRecon can reveal DNS records and software versions, and theHarvester can gather emails and hosts from public sources. In the scanning activity, I used Zenmap to run Quick and Intense scans, find open ports, services and the operating system of my PC, scan my gateway, read a traceroute and view the network topology.
The exercises showed me that information gathering is an important part of cybersecurity. Even before attempting to exploit a system, a security professional can learn a significant amount about an environment by carefully analyzing publicly available information and network responses. I also saw that tools can fail for reasons that have nothing to do with the target. My wafw00f run reported the site as down because of a name-resolution problem on my own machine. My theHarvester run with all sources returned nothing because many sources need API keys and some could not reach DNS. My first theHarvester command even had a typo in the
 
domain name. I learned to check my own setup and inputs before drawing conclusions, and that an empty result does not mean there is nothing to find.
I also learned that technical findings should be documented clearly. A good cybersecurity report should explain what was performed, what was discovered, what the observation means, what risk it may create, and what can be done to reduce that risk.
Finally, I learned that reconnaissance and scanning must always be performed within an authorized scope. The footprinting of networkwalks.com and the scans of my own network were completed under a written Letter of Authorization. The two practice targets, scanme.nmap.org and microsoft.com, are not named in that letter, so I have marked them clearly in this report and I will keep future testing inside the written scope.

8.	Evidences Collected
All screenshots are shown in Section 4 under the tool that produced them. The index below lists each one.

Figure	Description	Source file
Figure 1	WHOIS output for networkwalks.com (registry record)	Who_is.png
Figure 2	WHOIS output continued (registrar record begins)	who_is_2.png
Figure 3	WHOIS output continued (registrant, technical contact and name servers)	
who_is_3.png
Figure 4	WHOIS output continued (end of record)	who_is_4.png
Figure 5	WhatWeb output for networkwalks.com	WHATWEB.png
Figure 6	Nslookup output for networkwalks.com	NSLOOKUP.png
Figure 7	Curl -I output showing the HTTP response headers	CURL_.png
Figure 8	Wafw00f detecting ModSecurity (SpiderLabs)	WAFW00F_CONNECTED
_REPORT.png
Figure 9	Wafw00f run that failed because of a temporary DNS resolution error	
WAFWOOF.png
Figure 10	DNSRecon output for networkwalks.com	DNSRECON.png
Figure 11	theHarvester help screen (version 4.11.1)	HARVESTER_HOME_.pn
g
Figure 12	First run: Baidu source only, no results (domain typed as microsft.com)	HARVESTER_MICROSOF
T  COM.png
Figure 13	Second run with all sources: start of output and missing API key messages	HARVESTER_FROM_TE
RMINAL_1.png
Figure 14	Second run continued: more missing API key messages	TERMINAL_2.png
Figure 15	Second run continued: sources being searched, with several errors	
TERMINAL_3.png
Figure 16	End of the second run: no IPs, emails, people or hosts found	
TERMINAL_4.png
Figure 17	Zenmap Quick scan of 192.168.10.143	port_scanning_nmap.png
 

Figure	Description	Source file
Figure 18	Zenmap Intense scan of the gateway (192.168.10.1), partly masked	nmap_intense_scanning.p ng
Figure 19	Intense scan of 192.168.10.143: scan progress and open ports	
zenmap_143.png
Figure 20	Intense scan continued: OS detection, uptime and SMB script results	
zenmap_143  2.png
Figure 21	Ports / Hosts tab showing the six open ports and their versions	
zenmap_ports.png
Figure 22	Zenmap Topology tab with the legend open	zenmap_legend.png
Figure 23	Saved topology graphic	zenmap_graphics.png
Figure 24	Scan of scanme.nmap.org: scan progress	Trial_scan_nmap.png
Figure 25	Scan of scanme.nmap.org: open ports and service versions	SCAN_ME_TEST_SCAN_
NMAP.png
Figure 26	Scan of scanme.nmap.org: OS guesses and traceroute	SCAN_ME_TEST_SCAN_
NMAP_2.png
Figure 27	Topology graphic for the scanme.nmap.org traceroute	zenmap_TEST_SCAN.png
Figure 28	Letter of Authorization, page 1 of 2	W2-PM-
_Permission_Letter_v1.pdf
Figure 29	Letter of Authorization, page 2 of 2	W2-PM-
_Permission_Letter_v1.pdf

Written permission: Letter of Authorization


 
Figure 28: Letter of Authorization, page 1	
 
Figure 29: Letter of Authorization, page 2
- End -

Author
Ilemobayo John Oluwaseun
Cybersecurity Intern, Batch B084
Project Information
Program Name: Cybersecurity program at Networkwalks | Week: 02 | Repository: GitHub
