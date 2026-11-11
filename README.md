# 🛡️ Penetration Testing Report

### Footprinting & Network Scanning Phases

**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| | |
|---|---|
| **Pentester Name** (Cybersecurity Professional) | **[Humna Barira]** |
| **Program / Batch** | [B084E] - Networkwalks |
| **Date** | 11 November 2026 |
| **Modules completed** | W2-PM1 (Multiple Kali Tools) <br> W2-PM5 (Zenmap Scanning) |
| **Client / Target** | 1. Networkwalks (written permission already secured) <br> 2. My own local lab network |
| **Permission secured from client?** | Yes |
| **Phases covered** | **Phase 1:** Reconnaissance & Footprinting <br> **Phase 2:** Scanning & Network Discovery <br> **Phase 3-5:** In Progress |

---

## 📑 Table of Contents

1. [Liability Disclaimer](#1-liability-disclaimer)
2. [Introduction](#2-introduction)
3. [Tools Used](#3-tools-used)
4. [Activities Performed](#4-activities-performed)
5. [Risk Analysis / Impact](#5-risk-analysis--impact)
6. [Recommendations](#6-recommendations)
7. [Conclusion](#7-conclusion)
8. [Evidence Collected](#8-evidence-collected)

---

## 1. Liability Disclaimer

> ⚠️ I did these activities only on systems and devices where I had written permission, or on devices that I own.

All material here is for **education and research only**. Do not use anything from this repository to break the law.

The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries, unauthorised access is a crime, even when nothing is damaged.

---

## 2. Introduction

This report shows how I did **footprinting** on the `networkwalks.com` domain using Kali Linux tools (W2-PM1). It also shows how I **scanned my own lab network** with Zenmap (W2-PM5).

- **Footprinting** means collecting public information about a target.
- **Scanning** means finding live devices on a network.

Together, these two steps show how an attacker first gathers information and then maps the network.

This is the Week 2 part of my cybersecurity internship at Networkwalks. All commands were run on **Kali Linux**. For each step, I show:

- the exact command I used,
- the result I saw,
- a screenshot as proof,
- a short note on why it matters.

---

## 3. Tools Used

| Tool | Purpose |
|---|---|
| **Kali Linux** | Operating system used for all the work |
| **WHOIS** | Find domain registration details (owner, name servers) |
| **whatweb** | Find web technologies (server, CMS, plugins, IP) |
| **nslookup** | Turn a domain name into its IP address using DNS |
| **curl** | Read the HTTP response headers of a website |
| **wafw00f** | Check if a Web Application Firewall (WAF) protects the site |
| **dnsrecon** | List DNS records (NS, MX, SPF, TXT, SRV) |
| **theHarvester** | Collect emails, hosts and IPs from public (OSINT) sources |
| **ifconfig** | Find my local IP address and subnet |
| **Zenmap (Nmap GUI)** | Scan my local subnet to find live hosts, IPs and MAC addresses |

---

## 4. Activities Performed

### 4.1 Footprinting & Reconnaissance

I used seven tools on the `networkwalks.com` domain: **WHOIS, WhatWeb, Nslookup, Curl, Wafw00f, DNSRecon and theHarvester**. Each tool gives a different type of information.

#### 🔹 Step 1: WHOIS - Domain registration

```bash
whois networkwalks.com
```

**What I found:**
- The owner details are **hidden** by a privacy service (**Domains By Proxy, LLC**).
- Contact is only possible through a GoDaddy contact form.
- The domain status is `clientDeleteProhibited`, which means it is locked against deletion.

**Why it matters:** Privacy protection stops attackers from easily learning who owns the domain.

#### 🔹 Step 2: WhatWeb - Website technology

```bash
whatweb networkwalks.com
```

**What I found:**
- The first try over `http://` failed with *"execution expired"*. The `https://` version worked and returned **200 OK**.
- The site uses **WordPress** and the **WordPress Download Manager** plugin (version 3.3.76).
- Other items found: **Apache** web server, **jQuery 3.7.1**, **Bootstrap**, Google Tag Manager, HTML5.
- Server IP: **192.232.216.135**
- Public email: `info@networkwalks.com`
- Page title: **Networkwalks Academy**
- A cookie named `__wpdm_client` is set with the **HttpOnly** flag.

**Why it matters:** Knowing the CMS, plugin and version helps an attacker look for known problems.

#### 🔹 Step 3: Nslookup - Find the IP address

```bash
nslookup networkwalks.com
```

**What I found:** The domain points to **192.232.216.135** (answered by DNS server 8.8.8.8).

**Why it matters:** This is the network location of the web server.

#### 🔹 Step 4: Curl - HTTP headers

```bash
curl -i https://networkwalks.com
```

**What I found:**
- Response: **HTTP/2 200**
- Server: **Apache**
- Headers show a `permissions-policy` and a `referrer-policy`.
- The `link` header shows the WordPress REST API address: **`/wp-json/`**
- Extra headers `x-nginx-cache` and `x-endurance-cache-level` show caching is used.
- The cookie is set with `secure` and `HttpOnly`.

**Why it matters:** Headers can show server details and open endpoints that help with fingerprinting.

#### 🔹 Step 5: Wafw00f - Firewall detection

```bash
wafw00f networkwalks.com
```

**What I found:** The site is behind a **ModSecurity (SpiderLabs)** WAF. It took only 2 requests to find it.

**Why it matters:** An attacker now knows what security tool is in front of the site and can plan around it.

#### 🔹 Step 6: DNSRecon - DNS records

```bash
dnsrecon -d networkwalks.com
```

**What I found:**

| Record | Value |
|---|---|
| **SOA / NS** | `ns6135.hostgator.com` (50.87.144.87) and `ns6136.hostgator.com` (192.232.216.131) |
| **DNS software** | BIND version `9.16.23-RH` |
| **MX (mail)** | `mail.networkwalks.com` → 192.232.216.135 |
| **A** | `networkwalks.com` → 192.232.216.135 |
| **TXT** | Google site verification record |
| **SPF** | `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` |
| **SRV** | `_autodiscover._tcp` records pointing to `cpanelemaildiscovery.cpanel.net` (8 records) |

**Why it matters:** DNS records show the hosting company (HostGator), mail setup and server software. This helps build a map of the target.

#### 🔹 Step 7: theHarvester - Public information (OSINT)

```bash
theHarvester -d microsoft.com -l 1000 -b baidu
theHarvester -d microsoft.com -l 5 -b all
```

**What I found:**
- With the `baidu` source, the server disconnected and **no results** were returned.
- With `-b all`, some sources showed *"Missing API key"* messages (for example bevigil, bufferoverun, Censys).
- Even with limited sources, it found **71 IPs** and **1318 hosts**. No emails or people were found.

**Why it matters:** A lot of host information is public. Free tools can show how big a company's internet presence is, even without API keys.

> 📝 **Note:** theHarvester only reads public sources (passive OSINT). It does not touch the target systems directly. I used `microsoft.com` as a practice domain for this tool.

---

### 4.2 Network Scanning with Zenmap

For the second activity, I scanned my **own local lab network**. The goal was to find my IP address and subnet, discover live hosts, note their IP and MAC addresses, and draw a network topology.

#### 🔹 Step 1: Find my IP and subnet

```bash
ifconfig
```

**What I found:**
- Interface: `eth0`
- IP address: **10.0.0.0**
- Netmask: **255.255.255.0** (so the subnet is `10.0.0.0/24`)
- Broadcast: `10.0.0.255`
- MAC address: `08:00:27:63:b0:05` (a VirtualBox virtual machine)

#### 🔹 Step 2: Ping scan with Zenmap

I opened Zenmap, typed the subnet as the target and chose the **Ping scan** profile.

```bash
nmap -sn 10.0.0.0/24
```

**What I found:** Nmap scanned **256 IP addresses** and found **3 live hosts** in **2.36 seconds**.

| Host | MAC Address | Vendor / Note |
|---|---|---|
| `10.0.0.0` | - | My own Kali machine (localhost) |
| `10.0.0.1` | `52:54:00:12:35:00` | QEMU virtual NIC |
| `10.0.0.2` | `08:00:27:9B:6E:AB` | Oracle VirtualBox virtual NIC |

#### 🔹 Step 3: Network topology

I opened the **Topology** tab in Zenmap and turned on the **Legend**. The map shows my machine (`localhost`) in the centre, joined to `10.0.0.0`, `10.0.0.1` and `10.0.0.2`. All three hosts are **green**, which means they have fewer than 3 open ports, or no port scan was done (this was a ping scan only).

**Why it matters:** A simple ping scan quickly shows which devices are alive. Any unknown device on a network should be checked.

---

## 5. Risk Analysis / Impact

These are the possible risks based on the information I collected.

| # | Risk / Finding | Evidence | Potential Impact | Risk Level |
|---|---|---|---|---|
| 1 | Web technology exposed | WhatWeb found WordPress and WP Download Manager 3.3.76 | Attackers may use software and version details to look for known weaknesses | 🟡 Medium |
| 2 | Server IP address visible | Nslookup showed `192.232.216.135` | Shows the network location of the web server | 🟢 Low |
| 3 | HTTP technical details exposed | Curl showed headers and the `/wp-json/` endpoint | Helps with fingerprinting and further enumeration | 🟢 Low |
| 4 | WAF can be identified | Wafw00f found ModSecurity (SpiderLabs) | Reveals part of the site's security setup | 🟢 Low |
| 5 | DNS information exposed | DNSRecon showed NS, MX, SPF, SRV records and BIND version | Helps an attacker build a map of the infrastructure | 🟡 Medium |
| 6 | Public host and IP information | theHarvester found 71 IPs and 1318 hosts (practice domain) | Large public footprint is easier to study | 🟡 Medium |
| 7 | Live hosts visible on the local network | Zenmap found 3 live hosts | Unknown devices could be present on a network | 🟡 Medium |

**Risk level key:** 🔴 Critical &nbsp;&nbsp; 🟡 Medium &nbsp;&nbsp; 🟢 Low

> ℹ️ These are **observations**, not confirmed vulnerabilities.

I only gathered information and found live hosts. I did **not** exploit anything or test for vulnerabilities. Seeing a software version, an IP address or a DNS record does not mean a system is weak. More authorised testing is needed to confirm any real problem.

---

## 6. Recommendations

1. **Review what is public about your technology**
   Check regularly what your website, CMS and plugins show to the public.

2. **Keep software updated**
   Update WordPress, plugins and other tools often, and check them against security advisories.

3. **Review HTTP headers**
   Check whether your headers show more technical detail than they need to.

4. **Check DNS records regularly**
   Make sure only the records and services you need are public. Consider hiding the DNS software version.

5. **Keep the WAF on and tuned**
   Keep ModSecurity enabled and updated, because it already blocks simple attacks.

6. **Scan your own network often**
   Run internal discovery scans from time to time to know which devices are active.

7. **Investigate unknown devices**
   Check any device you do not recognise.

8. **Keep network documents up to date**
   Write down your network topology and device list, and update them.

9. **Test only with permission**
   Do reconnaissance and scanning only on systems where you have written authorisation.

---

## 7. Conclusion

In Week 2 of my Cybersecurity & Ethical Hacking internship, I did hands-on work in footprinting, reconnaissance and network scanning.

In the footprinting task, I used Kali Linux tools to collect information about a domain. I learned that:

- **WHOIS** shows domain registration details,
- **WhatWeb** finds web technologies,
- **Nslookup** turns names into IP addresses,
- **Curl** shows HTTP headers,
- **Wafw00f** finds a web application firewall,
- **DNSRecon** lists DNS records,
- **theHarvester** collects public information.

In the scanning task, I used Zenmap to find my network settings, discover live hosts, note their IP and MAC addresses, and see them on a topology map.

These exercises showed me that information gathering is a key part of cybersecurity. Before any attack, a person can learn a lot just from public information and network replies.

I also learned that findings must be written clearly. A good report explains what was done, what was found, what it means, what risk it creates, and how to reduce that risk.

Most important, reconnaissance and scanning must always stay inside an **authorised scope**. All of this work was done as part of an assigned educational lab.

---

## 8. Evidence Collected

### Footprinting

**1. WHOIS**

<img width="539" height="284" alt="01_whois" src="https://github.com/user-attachments/assets/49f04c13-8cee-40c5-b02f-dac0c0ed87c2" />


**2. WhatWeb**

<img width="772" height="65" alt="02_whatweb" src="https://github.com/user-attachments/assets/b689829d-6716-40ae-a751-90bbfd190724" />


**3. Nslookup**

<img width="201" height="83" alt="03_nslookup" src="https://github.com/user-attachments/assets/7b96226b-dff4-4be6-bbc1-2f6c5da21c0b" />


**4. Curl**

<img width="786" height="137" alt="04_curl" src="https://github.com/user-attachments/assets/62904909-389c-4f0e-86fd-10bd3656e72b" />

**5. Wafw00f**

<img width="361" height="173" alt="05_wafw00f" src="https://github.com/user-attachments/assets/8b8ae921-6f0e-45ea-bc2e-dcc53168c0bd" />


**6. DNSRecon**

<img width="584" height="219" alt="06_dnsrecon" src="https://github.com/user-attachments/assets/3233cd82-f2ca-49e7-8ce0-4681c2ed9307" />


**7. theHarvester**

<img width="318" height="182" alt="07_theharvester_baidu_error" src="https://github.com/user-attachments/assets/3d9e1039-6002-4a19-8a17-7795c797b51f" />

<img width="209" height="92" alt="08_theharvester_baidu_no_results" src="https://github.com/user-attachments/assets/8764acc5-0119-4359-a37b-3d044618c02c" />


<img width="419" height="297" alt="09_theharvester_all_sources" src="https://github.com/user-attachments/assets/ebf53011-9f43-4f9e-8f51-71bf5f47d59d" />

<img width="141" height="76" alt="10_theharvester_ips_found" src="https://github.com/user-attachments/assets/e41b0247-cbe0-48a3-b7a7-a08cd2bc3d06" />

<img width="265" height="52" alt="11_theharvester_hosts_found" src="https://github.com/user-attachments/assets/2d374e83-40b5-4e46-932c-85e5b66bf25c" />

### Network Scanning

**8. Zenmap start screen**

<img width="784" height="346" alt="12_zenmap_start" src="https://github.com/user-attachments/assets/be0e9f3d-3355-43ba-8418-ef601fa6a1c7" />


**9. Local IP and subnet (ifconfig)**

<img width="325" height="98" alt="13_ifconfig" src="https://github.com/user-attachments/assets/50375e5c-cfb7-4d2d-b857-6d3b4b170e0d" />


**10. Zenmap ping scan result**

<img width="783" height="325" alt="14_zenmap_ping_scan" src="https://github.com/user-attachments/assets/60b66480-4626-4680-ae10-fcd29351e30c" />


**11. Zenmap network topology**

<img width="787" height="349" alt="15_zenmap_topology" src="https://github.com/user-attachments/assets/8d8cc730-cdf1-4874-be5f-c4557d9578fa" />


---

*- End -*

