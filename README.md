## **PASSIVE RECONNAISSANCE - TryHackMe** 


|**Prepared by**|Parth|
|---|---|
|**Target Domain**|tryhackme.com (with thmlabs.com used for DNS TXT exercise)|
|**Engagement Type**|Open-Source Intelligence (OSINT) / Passive Reconnaissance|
|**Room**|TryHackMe — Passive Reconnaissance|
|**Date Completed**|June 20, 2026|



_Conducted entirely without direct interaction with target infrastructure, using only publicly available third-party data sources._ 


## **1. Executive Summary** 

This report documents a passive reconnaissance exercise performed as part of the TryHackMe “Passive Reconnaissance” room. The objective of the exercise was to demonstrate how an analyst can gather meaningful intelligence about a target domain — registration details, DNS configuration, hosting infrastructure, and exposed technologies — without ever sending traffic directly to the target's systems. 

All information was collected through third-party, publicly accessible data sources: WHOIS registries, DNS resolvers, certificate transparency / subdomain aggregators (DNSDumpster), and the Shodan internet-wide scanning platform. Because none of these techniques touch the target directly, they are undetectable by the target and form the foundation of any responsible reconnaissance methodology. 

The domain tryhackme.com was used as the primary subject for WHOIS, DNSDumpster, and Shodan exercises, while thmlabs.com was used for a DNS TXT record retrieval task that returned a unique flag, confirming successful command execution. 

## **2. Objective** 

- Identify domain ownership and registration metadata using WHOIS. 

- Query DNS records (MX, TXT) directly via nslookup / dig to understand mail routing and verify domain ownership artifacts. 

- Enumerate subdomains, hosting providers, and exposed service banners using DNSDumpster as a passive, certificate-transparency-backed aggregator. 

- Use Shodan.io to assess the global prevalence of common web server technologies (Apache, nginx) and the ports they are most commonly exposed on. 

- Document every command, tool, and result captured during the exercise in a single, reviewable report. 

## **3. Tools and Methodology Overview** 

The table below summarizes every tool used during the engagement, its purpose, and the interaction model (passive vs. direct). 

|**Tool**|**Purpose**|**Interacton Type**|
|---|---|---|
|whois|Domain registraton lookup (registrar, dates, name servers)|Passive (registry<br>query)|
|nslookup / dig|Direct DNS record queries (MX, TXT, A)|Passive (DNS query<br>only)|
|DNSDumpster|Subdomain discovery, hostng/network mapping, service<br>banner aggregaton|Passive (third-party<br>aggregator)|
|Shodan.io|Internet-wide scan data on technology prevalence, ports,<br>and geography|Passive (third-party<br>scan index)|

## **4. Task 1 — WHOIS Domain Lookup** 

## **4.1 Purpose of the Tool** 

WHOIS is a query/response protocol used to look up registration information held by domain registries and registrars. It reveals who registered a domain, when, through which registrar, and which name servers are authoritative for it — all without contacting the target's own servers. This is typically the first step of any reconnaissance engagement, establishing domain ownership context. 

## **4.2 Commands Executed** 

whois tryhackme.com 

## **4.3 Information Gathered** 

The use of Cloudflare-managed name servers indicates the domain's DNS is proxied/managed through Cloudflare, which also explains the prevalence of Cloudflare-related IP ranges and banners seen later in the DNSDumpster results. 


## **5. Task 2 — DNS Record Enumeration (MX & TXT)** 

## **5.1 Purpose of the Tool** 

nslookup and dig are command-line DNS resolution utilities used to query specific DNS record types directly from a resolver. Querying MX records reveals where a domain's email is routed, while TXT records often contain SPF/DKIM data, domain-verification tokens, or — in this lab — a unique flag used to confirm successful command execution. dig was used here in preference to nslookup as recommended, querying the public resolver 1.1.1.1 to avoid relying on a potentially logging ISP resolver. 

## **5.2 Commands Executed** 

dig tryhackme.com MX dig @1.1.1.1 thmlabs.com TXT 

## **5.3 Information Gathered** 

MX records for tryhackme.com — all mail is routed through Google Workspace (Gmail) infrastructure: 

|**Priority**|**Mail Exchange**|
|---|---|
|1|aspmx.l.google.com|
|5|alt1.aspmx.l.google.com|
|5|alt2.aspmx.l.google.com|
|10|alt3.aspmx.l.google.com|
|10|alt4.aspmx.l.google.com|



## **TXT record queried:** thmlabs.com 

**Resolver used:** 1.1.1.1 (Cloudflare public DNS) 

**Flag captured:** THM{a5b83929888ed36acb0272971e438d78} 


## **6. Task 3 — Passive Subdomain & Infrastructure Enumeration (DNSDumpster)** 

## **6.1 Purpose of the Tool** 

DNSDumpster is a free, passive reconnaissance platform that aggregates Certificate Transparency (CT) log data, historical DNS records, and other open-source signals to map a domain's subdomains, hosting providers, geographic server locations, and exposed service banners — all without sending any packets to the target itself. CT logs are particularly valuable because every publicly trusted TLS certificate issued since roughly 2015 is logged centrally, and the certificate's Subject Alternative Name (SAN) field frequently reveals subdomains that would otherwise be hard to discover. 

## **6.2 Commands / Actions Executed** 

Searched "tryhackme.com" on https://dnsdumpster.com 

## **6.3 Information Gathered** 

System hosting locations: United States, Ireland, India, and Canada — consistent with a globally distributed CDN/edge network (Cloudflare). 

Hosting / Network providers identified: 

- CLOUDFLARENET (Cloudflare, Inc.) 

- AMAZON-02 / AMAZON-AES (Amazon Web Services) 

- GOOGLE (Google LLC) 

- CLOUDFLARESPECTRUM 

Services / banners observed (by occurrence count) — this directly answers the question of which banner has the highest count: 

|**Service / Banner**|**Count**|
|---|---|
|cloudfare|30 (highest)|
|nginx/1.29.1|18|
|nginx/1.29.0|4|
|CloudFront|2|



Sample A record discovered: 

|**Host**|admin.tryhackme.com|
|---|---|
|**IP Address**|104.20.29.66|
|**ASN**|13335 (CLOUDFLARENET — Cloudfare, Inc.)|

The “Direct IP access not allowed” banner confirms that origin servers are shielded behind Cloudflare's reverse proxy — direct connections to the disclosed IP do not reach the real application, reinforcing the value of passive (rather than active) reconnaissance against this target. 

## **7. Task 4 — Shodan.io Internet-Wide Scan Analysis** 

## **7.1 Purpose of the Tool** 

Shodan is a search engine for internet-connected devices and services. Rather than crawling web page content like a conventional search engine, Shodan continuously scans the public IPv4/IPv6 space and indexes service banners, open ports, and software versions. This makes it possible to passively assess how prevalent a given technology is worldwide, which countries host the most instances, and which ports it is typically exposed on — all without scanning the target organization directly. 

## **7.2 Commands / Actions Executed** 

Searched "apache" on https://www.shodan.io Searched "nginx" on https://www.shodan.io 

## **7.3 Information Gathered** 

Apache search results: 

|**Total Results**|13,981,292 publicly accessible Apache servers|
|---|---|
|**Top Country**|United States — 4,022,085 hosts|
|**Other Top Countries**|Germany, Japan, China, France|
|**Top Ports**|80, 443, 8080, 8443, 8081|
|**3rd Most Common**<br>**Apache Port**|8080|


nginx search results: 

|nginx search results:||
|---|---|
|**Total Results**|49,772,156 publicly accessible nginx servers|
|**Top Countries**|United States, China, Hong Kong, Japan, Germany|
|**Top Ports**|80, 443, 5001, 5000, 8888|
|**Most Common nginx**<br>**Port**|80|


The United States leads in both Apache and nginx deployments by a wide margin, and HTTP/HTTPS (ports 80/443) dominate as expected, with secondary ports (8080, 8443, 5000-series) reflecting common alternative HTTP, proxy, and admin-panel configurations. 


## **8. Key Findings Summary** 

|**#**|**Finding**|**Source**|
|---|---|---|
|1|tryhackme.com registered 2018-07-05 via Namecheap; DNS managed by<br>Cloudfare|WHOIS|
|2|Email routed through Google Workspace (5 MX records, aspmx.l.google.com<br>family)|dig (MX)|
|3|thmlabs.com TXT fag: THM{a5b83929888ed36acb0272971e438d78}|dig (TXT)|
|4|Infrastructure spans Cloudfare, AWS, and Google; origin IPs shielded behind<br>Cloudfare proxy|DNSDumpster|
|5|Apache: ~14M public instances worldwide, US-led, port 8080 is 3rd most<br>common|Shodan|
|6|nginx: ~50M public instances worldwide, US-led, port 80 is most common|Shodan|



## **9. Conclusion** 

This exercise reinforced the core principle of passive reconnaissance: substantial, actionable intelligence about a target's ownership, DNS configuration, hosting footprint, and exposed technologies can be gathered entirely through third-party data sources — WHOIS registries, DNS resolvers, certificatetransparency aggregators, and internet-wide scan indexes — without the target ever observing a single request from the analyst. This intelligence forms the foundation for any subsequent, more invasive phase of a security assessment and should always be exhausted first, both for stealth and for efficiency. 

All TryHackMe room questions across the WHOIS, DNS, DNSDumpster, and Shodan tasks were answered correctly, as evidenced by the green “Correct Answer” confirmations captured in the accompanying screenshots. 

