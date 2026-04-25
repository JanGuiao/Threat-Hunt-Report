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
How many externally sourced authentication events related to Remote Desktop were recorded?

Format: Number only

### Evidence
It is worth noting that in this dataset RDP auth volume shows both Network and RemoteInteractive logon types together - not just RemoteInteractive alone.
<img width="899" height="354" alt="image" src="https://github.com/user-attachments/assets/167f6781-6c5b-4f41-8faa-0aae3d7a81e2" />

### Answer
675


### Q13 - Dominant Auth Outcome

### Objective
Which authentication outcome was most frequently recorded for RDP-related logon activity?

Format: ActionType value

### Evidence
Of 675 external RDP-related authentication events, 646 were LogonFailed and only 29 were LogonSuccess! This tells us that there is an overwhelming failure rate confirms sustatined brute-force credential stuffing against the exposed RDP service.

<img width="899" height="433" alt="image" src="https://github.com/user-attachments/assets/e0876af4-ee8d-4fa1-ac2d-6f348f7f1947" />

### Answer
LogonFailed


### Q14 - Dominant Failure Reason

### Objective
What was the most common failure reason recorded for RDP-related authentication attempts?

Format: FailureReason value

### Evidence
Of 646 failed external RDP authentication attempts 637 failed due to InvalidUserNameOrPassword; indicating brute-force/credential stuffing behavior.
<img width="899" height="429" alt="image" src="https://github.com/user-attachments/assets/e118ebdb-8f0e-4538-8458-e1d64f9dd5e7" />

### Answer
InvalidUserNameOrPassword


### Q15 - Countries from Auth Activity

### Objective
How many unique countries were associated with RDP-related authentication events for the device?

Format: Number only

### Evidence
RDP-related authentication attempts originated from 17 distinct countries, confirmed via GeoIP enrichment using the briefing's reusable GeoTable snippet.
<img width="894" height="368" alt="image" src="https://github.com/user-attachments/assets/8d221dbf-ee11-4b87-b7f9-af64ae989670" />


### Answer
17


### Q16 - Countries with Successful Auth

### Objective
Of those countries, how many had at least one successful authentication event?

Format: Number only

### Evidence
17 countries associated with RDP authentication activity, only 2 had at least one successful logon. This is a critical finding - brute-force attempts from 17 countries, but successful authentication achieved from just 2. To isolate this, we scoped the same GeoIP-enriched query from Q15 but added a single filter — where ActionType == "LogonSuccess" — narrowing from all 675 RDP auth events down to just the 29 successful ones. The dcount(country_name) aggregation then counted distinct countries remaining after that filter, giving us the 2 countries that successfully authenticated rather than just attempted. The GeoTable externaldata() block from the briefing was prepended as before to perform the CIDR-based geographic enrichment on the RemoteIP field.
<img width="896" height="362" alt="image" src="https://github.com/user-attachments/assets/372eea18-189d-41e9-82fe-7884ca2c3682" />

### Answer
2


### Q17 - Successful Countries

### Objective
Which countries were associated with successful RDP authentication events?

Format: Country 1, Country 2

### Evidence
To identify this, we extended the Q16 query by replacing dcount(country_name) with summarize count() by country_name — shifting from counting distinct countries to showing each country alongside its successful logon volume. This gave us both the geographic origin and the weight of activity per country, revealing Uruguay as the dominant source of successful authentications.
<img width="898" height="385" alt="image" src="https://github.com/user-attachments/assets/981936bd-f18f-4eac-beb4-e9e17d8d4371" />

### Answer
Uruguay, United States

### Q18 - Unexpected Country

### Objective
Which account was used in the successful RDP authentication originating from the unexpected country?

Format: AccountName
### Evidence
PHTG operates exclusively in the United States, making the 23 successful RDP authentications from Uruguay entirely outside expected operational baseline. The United States activity (6 successes) could reasonably be attributed to legitimate administrative access, but Uruguay has no business justification whatsoever - WITH 23 successful logons from Uruguay.
<img width="898" height="385" alt="image" src="https://github.com/user-attachments/assets/1330e98d-7eef-46d2-bde0-9f870df337aa" />


### Answer
Uruguay


### Q19 - Account Used

### Objective
Which account was used in the successful RDP authentication originating from the unexpected country?

Format: AccountName
### Evidence
The account used in all successful RDP authentications originating from Uruguay was vmadminusername — a generic, predictable administrator account name. The account name itself suggests it was a default or lazily named admin account on the VM, making it trivial for automated credential stuffing tools to guess. This confirms the attacker successfully brute-forced their way in via RDP using this account.

To identify this, we restructured the query into two let blocks to avoid the KQL syntax limitation of nesting let inside a subquery. The first block (GeoTable) loaded the GeoIP dataset, the second (UruguayIPs) filtered successful logons and geo-matched them to Uruguay, extracting distinct source IPs. The final query then joined back to DeviceLogonEvents on those IPs to retrieve the AccountName field.
<img width="898" height="393" alt="image" src="https://github.com/user-attachments/assets/8bf75347-2658-4be3-8e52-ada80fe4ee56" />

### Answer
vmadminusername



### Q20 - Uruguay Success Count

### Objective
How many successful RDP authentication events originated from the unexpected country?

Format: Number only

### Evidence
Going back to the Q17 query, results already revealed that 23 successful RDP authentication events originated from Uruguay. 

<img width="898" height="385" alt="image" src="https://github.com/user-attachments/assets/981936bd-f18f-4eac-beb4-e9e17d8d4371" />

### Answer
23


### Q21 - First RemoteIP from Uruguay

### Objective
What is the RemoteIP associated with the first successful RDP authentication from the unexpected country?

Format: IPv4 address

### Evidence
The first successful RDP authentication from Uruguay occurred on 2025-12-12 at 05:47 UTC from IP 173.244.55.131. This is significant — the LinkedIn post exposed the VM on 12/11, and by 12/12 an attacker had already successfully brute-forced their way in via RDP.

To identify this, we took the GeoIP-enriched query from Q20, filtered to Uruguay successful logons, sorted by TimeGenerated asc and used take 1 to isolate the earliest event, projecting just the timestamp and source IP.

<img width="892" height="395" alt="image" src="https://github.com/user-attachments/assets/b9f79907-14c2-4674-8515-60ea694e783c" />

### Answer
173.244.55.131


### Q22 - Second RemoteIP from Uruguay

### Objective
Besides the RemoteIP from Q21, what is the other RemoteIP associated with successful authentication events from the unexpected country?

### Evidence
The second Uruguay-associated IP was 173.244.55.128 — and critically, both IPs (173.244.55.131 and 173.244.55.128) fall within the same /24 subnet (173.244.55.0/24). This is a strong indicator of a single threat actor operating across multiple IPs within the same infrastructure block, likely rotating IPs to evade simple IP-based blocking while maintaining persistent RDP access to the VM.

To identify this, we reused the Q21 query and added where RemoteIP != "173.244.55.131" to exclude the first known IP, then used distinct RemoteIP to surface the remaining unique source address from the Uruguay successful logons.
<img width="889" height="359" alt="image" src="https://github.com/user-attachments/assets/f47a1640-3827-46c1-a4d9-8d84a16583bd" />

### Answer
173.244.55.128


### Q23 - First Notable Process

### Objective
Session startup is noisy. After the first successful Uruguay authentication, routine processes run before any operator behaviour. Identify the first notable process ... the earliest one that indicates a person began interacting with the system purposefully.

Format: FileName

### Evidence
After session startup noise (DismHost.exe at 05:48 UTC launched by cleanmgr.exe /autoclean), the first notable process indicating purposeful human operator interaction was notepad.exe, launched at 2025-12-12 16:37:41 UTC under vmadminusername, spawned directly by:
"powershell.exe" -NoProfile -ExecutionPolicy Bypass -File "C:\Lab\SarahChen_Activity.ps1"

The -NoProfile -ExecutionPolicy Bypass flags are classic attacker PowerShell execution patterns — bypassing execution policy restrictions while avoiding profile scripts that might alert or log. The script name SarahChen_Activity.ps1 directly references the cloud engineer whose LinkedIn post exposed the VM, suggesting the attacker was either mimicking legitimate user activity or had access to PHTG's internal lab scripts.

<img width="892" height="527" alt="image" src="https://github.com/user-attachments/assets/a733b38e-e56f-4eb0-845e-392adc7850fb" />

### Answer
notepad.exe



### Q24 - Sensitive Text File

### Objective
Multiple text files were opened during the session. Which one contains internal security-relevant content that would meaningfully reduce an attacker's effort if reviewed?

Format: FileName
### Evidence
(C:\Users\vmAdminUsername\Documents\PHTG\notes_sarah.txt) stands out as the most security-relevant. The file is named after Sarah Chen — the same cloud engineer whose LinkedIn post exposed the VM's public IP and triggered this entire investigation. Her personal notes stored on the VM would likely contain infrastructure details, credentials, service configurations, or internal operational context. 

<img width="885" height="495" alt="image" src="https://github.com/user-attachments/assets/1db02eef-d543-4f72-a5f1-dd5947afeb32" />


### Answer
notes_sarah.txt


### Q25 - Successful Countries

### Objective

### Evidence

### Answer


### Q26 - Successful Countries

### Objective

### Evidence

### Answer

### Q27 - Successful Countries

### Objective

### Evidence

### Answer



### Q28 - Successful Countries

### Objective

### Evidence

### Answer


### Q29 - Successful Countries

### Objective

### Evidence

### Answer



### Q30 - Successful Countries

### Objective

### Evidence

### Answer



### Q31 - Successful Countries

### Objective

### Evidence

### Answer



### Q32 - Successful Countries

### Objective

### Evidence

### Answer


### Q33 - Successful Countries

### Objective

### Evidence

### Answer



### Q34 - Successful Countries

### Objective

### Evidence

### Answer



### Q35 - Successful Countries

### Objective

### Evidence

### Answer



### Q36 - Successful Countries

### Objective

### Evidence

### Answer


### Q37 - Successful Countries

### Objective

### Evidence

### Answer


### Q38 - Successful Countries

### Objective

### Evidence

### Answer


### Q17 - Successful Countries

### Objective

### Evidence

### Answer

### Q17 - Successful Countries

### Objective

### Evidence

### Answer

