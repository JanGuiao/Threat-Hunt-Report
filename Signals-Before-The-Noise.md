# Threat Hunt Report: Signals Before The Noise 
#### Author: Jan Guiao
#### Date: April 2026

## Platforms and Languages Used

### Platforms/Workspace:
* Microsoft Sentinel
* Logs Analytics Workspace (law-cyber-range)

### Languages/Tools:
* Kusto Query Language (KQL) for querying device events, registry search, and persistence.

## General:
Someone on the cloud team posted a photo on LinkedIn. Workstation, Azure portal open, VM details on screen. Could be nothing. Could be everything."

"I need you to figure out what was exposed, whether anyone noticed, and whether anyone acted on it. Start with the images. Extract what you can. Then pivot to telemetry. If someone used what was in that photo to get in, I want to know about it before they come back."

## Objective: 

There is no incident. There are no alerts. There is no suspected compromise. Yet. Review the publicly visible evidence. Determine what was exposed. Investigate whether the exposure was exploited. Follow the telemetry wherever it leads.

## Flag Analysis
<details>
<summary id="-flag-1">🚩 <strong>Flag 1: <Technique Name></strong></summary>  

### Q01 - Identifying the Exposed Asset

### Objective 
You are reviewing what has been publicly visible about PHTG's cloud infrastructure. The screenshot below shows one of the company's Azure virtual machines from the management portal.

Anchor the investigation to a specific resource. What is the name of this virtual machine?


Format: Hostname

### Evidence
Clear as day from the Azure Portal screenshot- the hostname or VM name is azwks-phtg-02.

<img width="1363" height="996" alt="image" src="https://github.com/user-attachments/assets/08bdc09d-7d5f-44ea-b236-21fb5c23b5b9" />


### Answer
awks-phtg-02


### Q02 - Public Exposure Vector

### Objective 
A virtual machine existing in Azure is not, by itself, a problem. A virtual machine reachable from the open internet is.

Review the networking details in the portal screenshot. What public IP address is currently associated with this VM?


Format: IPv4 address

### Evidence
Right there in the Networking section of the screnshot is where the Public IP Address is. Its worth addressing that their is no DNS name configured, i.e. sitting exposed directly by IP.

<img width="1363" height="996" alt="image" src="https://github.com/user-attachments/assets/08bdc09d-7d5f-44ea-b236-21fb5c23b5b9" />

### Answer
74.249.82.162


###  Q03 - When Context Becomes Actionable

### Objective 
The VM is exposed to the internet. But exposure alone does not tell you what an attacker gains from seeing this view. Focus on the networking panel.

What specifically gives an external observer the clearest indication that a threat actor could act on this information?

A. The virtual machine is running Windows 10 Enterprise B. The VM size and memory configuration are displayed C. The VM is located in a specific cloud region D. A public IP address is visible and associated with the virtual machine E. The VM has tags applied for management

Format: Single letter

### Evidence
A public IP address directly associated with the VM is the most likely the identifying factor when threat actors take action-- considering the other pieces (OS, size, region, tags) are also context- but a public IP is an invitation. Any form of reconnaissance, port scanning or exploitation attempts can veryh much well start against that address.


### Answer
D

### Q04 - OSINT Correlation

### Objective
Q04 - OSINT Correlation
50
The Azure portal view does not leak itself. Someone shared it.

A colleague in PHTG's cloud engineering team recently posted about their first day on a new internal service. The attached photo shows their workstation. Reviewing what is on the screen, what type of activity best describes what is being performed?


A. Writing application source code B. Responding to a security incident C. Managing cloud infrastructure resources D. Monitoring system performance dashboards E. Reviewing internal documentation

### Evidence
The monitor shows the Azure portal and what looks like a terminal/log output. Looking back into the briefing "Workstation, Azure portal open, VM details on screen." matches the description perfectly.

<img width="656" height="376" alt="image" src="https://github.com/user-attachments/assets/7fbfcdac-d728-4bfb-a55f-e259d416b11c" />

### Answer
C


### Q05 - Evidence Source Selection

### Objective
The VM has a public IP. That does not mean it was reached, targeted, or interacted with. Only that the barrier is lower. Which telemetry source would you review first to determine whether the exposed public IP was subject to scanning or enumeration?

A. Windows Application Event Logs B. Azure Active Directory sign-in logs C. Microsoft Defender for Endpoint device inventory D. Azure network or platform analytics related to inbound connections E. Local browser history on the virtual machine

### Evidence
Azure network network or platform analytics would show inbound connection attempts to public IP- this includes things like scans, and enumerations before anything ever touches the operating system (OS). It is an external/outermost visible layer.

### Answer
D


### Q06 - Broad Scanning Indicators

### Objective

Using MDE DeviceNetworkEvents, which local port shows the strongest indicator of broad, automated scanning against the VM?

Format: Port number

### Evidence

194 Inbound connections attempts against port 3389 over 11 days, starting just hours after the LinkedIN post went up (VM was created 12/10, HealthCloud rolled out 12/11, and scanning begins 12/11 at 03:42 UTC).

<img width="898" height="457" alt="image" src="https://github.com/user-attachments/assets/6977fd2a-7a95-4d4f-b398-aba44e7fb469" />

### Answer 
3389

### Q07 - Exposure Activity Volume

### Objective
How many network events targeted the port identified in Q06 on the device?

Format: Number only

### Evidence
Regarding the previous (Q06), we can see that 194 inbound connection attempts against port 3389 over the span of "11 days"
<img width="903" height="454" alt="image" src="https://github.com/user-attachments/assets/e958f54d-09af-4bd5-a403-49461ef7b105" />

### Answer
194


### Q08 - Source Diversity

### Objective
How many unique public source IP addresses targeted the exposed service?

Format: Number only

### Evidence
173 unique public IPs attacking via port 3389. This can also be an indicator of automated, distributed scanning- i.e. a broad sweep from possible botnets. 
<img width="898" height="365" alt="image" src="https://github.com/user-attachments/assets/0b240d2b-59b7-4515-84f9-5f6317cc0b5d" />

### Answer
173

### Q09 - Connection Outcomes

### Objective
How many source IPs show both a connection attempt and an accepted connection against the exposed service? Scanners that get a TCP response are a different class than raw probes.

Format: Number only

### Evidence
57 IPs showed both a ConnectionAttempt AND an InboundConnectionAccepted on port 3389 - these are the ones that got a TCP response back.

<img width="903" height="355" alt="image" src="https://github.com/user-attachments/assets/ed32285b-1b00-4a5e-8c6c-6c2f22ae0dc9" />


### Answer
57

### Q10 - Countries with RDP Activity

### Objective
Take the sources from Q09 and enrich them with geographic data. How many distinct countries are associated with that RDP connection activity?

Use the GeoTable snippet from the briefing.

Format: Number only

### Evidence
Having identified 57 IPs that showed both a ConnectionAttempt and InboundConnectionAccepted on port 3389, the next step was to determine where this activity was originating from. Raw IP addresses alone provide no geographic context, so we enriched the dataset using a GeoIP lookup.
The hunt briefing provided two key pieces that made this possible:

The GeoTable reusable snippet — listed explicitly in the briefing under Reusable Snippets // GeoIP Lookup, this KQL block uses Sentinel's externaldata() operator to pull a publicly available GeoIP CIDR dataset hosted on GitHub (datasets/geoip2-ipv4) at query time — no prior import into the workspace required.
The ipv4_lookup() guidance — the briefing instructed analysts to "prepend this block to your query, then pipe your IP column through evaluate ipv4_lookup(GeoTable, RemoteIP, network)" — giving us the exact join method to match each IP against its CIDR range and return country-level metadata.

<img width="902" height="356" alt="image" src="https://github.com/user-attachments/assets/a72198fb-d8fa-454c-ab29-a6a870b0e2bc" />


### Answer
11

### Q11 - Total External Auth Volume

### Objective
How many externally sourced authentication events were recorded for the device?

Format: Number only

### Evidence
This pulls all authentication events originating from public/external IPs against the device.
<img width="904" height="371" alt="image" src="https://github.com/user-attachments/assets/02865bb0-f0ba-4f59-aee3-865a27a34067" />

### Answer
693


### Q12 - RDP Auth Volume 

### Objective

### Evidence

### Answer



