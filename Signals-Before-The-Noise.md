# Threat Hunt Report: Signals Before The Noise 
#### Participant: Jan Guiao
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

## Scenario/Finding/Goal: 

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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 2: <Technique Name></strong></summary>  
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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 3: <Technique Name></strong></summary>  
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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 4: <Technique Name></strong></summary>  
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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 5: <Technique Name></strong></summary>  
### Q05 - Evidence Source Selection

### Objective
The VM has a public IP. That does not mean it was reached, targeted, or interacted with. Only that the barrier is lower. Which telemetry source would you review first to determine whether the exposed public IP was subject to scanning or enumeration?

A. Windows Application Event Logs B. Azure Active Directory sign-in logs C. Microsoft Defender for Endpoint device inventory D. Azure network or platform analytics related to inbound connections E. Local browser history on the virtual machine

### Evidence
Azure network network or platform analytics would show inbound connection attempts to public IP- this includes things like scans, and enumerations before anything ever touches the operating system (OS). It is an external/outermost visible layer.

### Answer
D
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 6: <Technique Name></strong></summary>  
### Q06 - Broad Scanning Indicators

### Objective

Using MDE DeviceNetworkEvents, which local port shows the strongest indicator of broad, automated scanning against the VM?

Format: Port number

### Evidence

194 Inbound connections attempts against port 3389 over 11 days, starting just hours after the LinkedIN post went up (VM was created 12/10, HealthCloud rolled out 12/11, and scanning begins 12/11 at 03:42 UTC).

<img width="898" height="457" alt="image" src="https://github.com/user-attachments/assets/6977fd2a-7a95-4d4f-b398-aba44e7fb469" />

### Answer 
3389
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 7: <Technique Name></strong></summary>  
### Q07 - Exposure Activity Volume

### Objective
How many network events targeted the port identified in Q06 on the device?

Format: Number only

### Evidence
Regarding the previous (Q06), we can see that 194 inbound connection attempts against port 3389 over the span of "11 days"
<img width="903" height="454" alt="image" src="https://github.com/user-attachments/assets/e958f54d-09af-4bd5-a403-49461ef7b105" />

### Answer
194
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 8: <Technique Name></strong></summary>  
### Q08 - Source Diversity

### Objective
How many unique public source IP addresses targeted the exposed service?

Format: Number only

### Evidence
173 unique public IPs attacking via port 3389. This can also be an indicator of automated, distributed scanning- i.e. a broad sweep from possible botnets. 
<img width="898" height="365" alt="image" src="https://github.com/user-attachments/assets/0b240d2b-59b7-4515-84f9-5f6317cc0b5d" />

### Answer
173
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 9: <Technique Name></strong></summary>  
### Q09 - Connection Outcomes

### Objective
How many source IPs show both a connection attempt and an accepted connection against the exposed service? Scanners that get a TCP response are a different class than raw probes.

Format: Number only

### Evidence
57 IPs showed both a ConnectionAttempt AND an InboundConnectionAccepted on port 3389 - these are the ones that got a TCP response back.

<img width="903" height="355" alt="image" src="https://github.com/user-attachments/assets/ed32285b-1b00-4a5e-8c6c-6c2f22ae0dc9" />


### Answer
57
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 10: <Technique Name></strong></summary>  
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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 11: <Technique Name></strong></summary>  
### Q11 - Total External Auth Volume

### Objective
How many externally sourced authentication events were recorded for the device?

Format: Number only

### Evidence
This pulls all authentication events originating from public/external IPs against the device.
<img width="904" height="371" alt="image" src="https://github.com/user-attachments/assets/02865bb0-f0ba-4f59-aee3-865a27a34067" />

### Answer
693
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 12: <Technique Name></strong></summary>  
### Q12 - RDP Auth Volume 

### Objective
How many externally sourced authentication events related to Remote Desktop were recorded?

Format: Number only

### Evidence
It is worth noting that in this dataset RDP auth volume shows both Network and RemoteInteractive logon types together - not just RemoteInteractive alone.
<img width="899" height="354" alt="image" src="https://github.com/user-attachments/assets/167f6781-6c5b-4f41-8faa-0aae3d7a81e2" />

### Answer
675
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 13: <Technique Name></strong></summary>  
### Q13 - Dominant Auth Outcome

### Objective
Which authentication outcome was most frequently recorded for RDP-related logon activity?

Format: ActionType value

### Evidence
Of 675 external RDP-related authentication events, 646 were LogonFailed and only 29 were LogonSuccess! This tells us that there is an overwhelming failure rate confirms sustatined brute-force credential stuffing against the exposed RDP service.

<img width="899" height="433" alt="image" src="https://github.com/user-attachments/assets/e0876af4-ee8d-4fa1-ac2d-6f348f7f1947" />

### Answer
LogonFailed
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 14: <Technique Name></strong></summary>  
### Q14 - Dominant Failure Reason

### Objective
What was the most common failure reason recorded for RDP-related authentication attempts?

Format: FailureReason value

### Evidence
Of 646 failed external RDP authentication attempts 637 failed due to InvalidUserNameOrPassword; indicating brute-force/credential stuffing behavior.
<img width="899" height="429" alt="image" src="https://github.com/user-attachments/assets/e118ebdb-8f0e-4538-8458-e1d64f9dd5e7" />

### Answer
InvalidUserNameOrPassword
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 15: <Technique Name></strong></summary>  
### Q15 - Countries from Auth Activity

### Objective
How many unique countries were associated with RDP-related authentication events for the device?

Format: Number only

### Evidence
RDP-related authentication attempts originated from 17 distinct countries, confirmed via GeoIP enrichment using the briefing's reusable GeoTable snippet.
<img width="894" height="368" alt="image" src="https://github.com/user-attachments/assets/8d221dbf-ee11-4b87-b7f9-af64ae989670" />


### Answer
17
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 16: <Technique Name></strong></summary>  
### Q16 - Countries with Successful Auth

### Objective
Of those countries, how many had at least one successful authentication event?

Format: Number only

### Evidence
17 countries associated with RDP authentication activity, only 2 had at least one successful logon. This is a critical finding - brute-force attempts from 17 countries, but successful authentication achieved from just 2. To isolate this, we scoped the same GeoIP-enriched query from Q15 but added a single filter — where ActionType == "LogonSuccess" — narrowing from all 675 RDP auth events down to just the 29 successful ones. The dcount(country_name) aggregation then counted distinct countries remaining after that filter, giving us the 2 countries that successfully authenticated rather than just attempted. The GeoTable externaldata() block from the briefing was prepended as before to perform the CIDR-based geographic enrichment on the RemoteIP field.
<img width="896" height="362" alt="image" src="https://github.com/user-attachments/assets/372eea18-189d-41e9-82fe-7884ca2c3682" />

### Answer
2
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 17: <Technique Name></strong></summary>  
### Q17 - Successful Countries

### Objective
Which countries were associated with successful RDP authentication events?

Format: Country 1, Country 2

### Evidence
To identify this, we extended the Q16 query by replacing dcount(country_name) with summarize count() by country_name — shifting from counting distinct countries to showing each country alongside its successful logon volume. This gave us both the geographic origin and the weight of activity per country, revealing Uruguay as the dominant source of successful authentications.
<img width="898" height="385" alt="image" src="https://github.com/user-attachments/assets/981936bd-f18f-4eac-beb4-e9e17d8d4371" />

### Answer
Uruguay, United States
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 18: <Technique Name></strong></summary>  
### Q18 - Unexpected Country

### Objective
Which account was used in the successful RDP authentication originating from the unexpected country?

Format: AccountName
### Evidence
PHTG operates exclusively in the United States, making the 23 successful RDP authentications from Uruguay entirely outside expected operational baseline. The United States activity (6 successes) could reasonably be attributed to legitimate administrative access, but Uruguay has no business justification whatsoever - WITH 23 successful logons from Uruguay.
<img width="898" height="385" alt="image" src="https://github.com/user-attachments/assets/1330e98d-7eef-46d2-bde0-9f870df337aa" />


### Answer
Uruguay
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 19: <Technique Name></strong></summary>  
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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 20: <Technique Name></strong></summary>  
### Q20 - Uruguay Success Count

### Objective
How many successful RDP authentication events originated from the unexpected country?

Format: Number only

### Evidence
Going back to the Q17 query, results already revealed that 23 successful RDP authentication events originated from Uruguay. 

<img width="898" height="385" alt="image" src="https://github.com/user-attachments/assets/981936bd-f18f-4eac-beb4-e9e17d8d4371" />

### Answer
23
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 21: <Technique Name></strong></summary>  
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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 22: <Technique Name></strong></summary>  
### Q22 - Second RemoteIP from Uruguay

### Objective
Besides the RemoteIP from Q21, what is the other RemoteIP associated with successful authentication events from the unexpected country?

### Evidence
The second Uruguay-associated IP was 173.244.55.128 — and critically, both IPs (173.244.55.131 and 173.244.55.128) fall within the same /24 subnet (173.244.55.0/24). This is a strong indicator of a single threat actor operating across multiple IPs within the same infrastructure block, likely rotating IPs to evade simple IP-based blocking while maintaining persistent RDP access to the VM.

To identify this, we reused the Q21 query and added where RemoteIP != "173.244.55.131" to exclude the first known IP, then used distinct RemoteIP to surface the remaining unique source address from the Uruguay successful logons.
<img width="889" height="359" alt="image" src="https://github.com/user-attachments/assets/f47a1640-3827-46c1-a4d9-8d84a16583bd" />

### Answer
173.244.55.128
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 23: <Technique Name></strong></summary>  
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
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 24: <Technique Name></strong></summary>  
### Q24 - Sensitive Text File

### Objective
Multiple text files were opened during the session. Which one contains internal security-relevant content that would meaningfully reduce an attacker's effort if reviewed?

Format: FileName
### Evidence
(C:\Users\vmAdminUsername\Documents\PHTG\notes_sarah.txt) stands out as the most security-relevant. The file is named after Sarah Chen — the same cloud engineer whose LinkedIn post exposed the VM's public IP and triggered this entire investigation. Her personal notes stored on the VM would likely contain infrastructure details, credentials, service configurations, or internal operational context. 

<img width="885" height="495" alt="image" src="https://github.com/user-attachments/assets/1db02eef-d543-4f72-a5f1-dd5947afeb32" />

### Answer
notes_sarah.txt
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 25: <Technique Name></strong></summary>  
### Q25 - First Executable Form

### Objective
Looking at FileRenamed events for the payload file, what is the first renamed filename where the extension turns the file into a Windows executable?

Format: FileName

### Evidence
The first file rename event converting a file into a Windows executable occurred on 2025-12-12 at 14:18:38 UTC. The file Sarah_Chen_Notes.exe.Txt was renamed to Sarah_Chen_Notes.exe in C:\Users\vmAdminUsername\Documents\PHTG\, initiated by explorer.exe under vmadminusername.

To identify this, we filtered DeviceFileEvents for ActionType == "FileRenamed" with executable extensions, excluding the system account and SoftwareDistribution path to eliminate Windows Update noise, leaving only attacker-controlled file activity under vmadminusername.

<img width="886" height="503" alt="image" src="https://github.com/user-attachments/assets/41c4fa82-8290-4b7f-95e3-951d8afd82cd" />

### Answer
Sarah_Chen_Notes.exe > .exe.Txt
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 26: <Technique Name></strong></summary>  
### Q26 - Double-Extension Evasion

### Objective
Between the file arriving as a text file and running as an executable, the attacker briefly used a filename with two extensions. What was it?

Format: FileName
### Evidence
Prior to execution, the payload existed under the filename Sarah_Chen_Notes.exe.Txt — a double-extension file. This technique takes advantage of Windows hiding known file extensions by default, meaning the file would appear as Sarah_Chen_Notes.exe to a user browsing the directory, concealing its true .Txt extension.
<img width="885" height="491" alt="image" src="https://github.com/user-attachments/assets/8965905f-7fc5-4447-8247-e9e0ca480f60" />

### Answer
Sarah_Chen_Notes.exe.Txt
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 27: <Technique Name></strong></summary>  
### Q27 - File SHA256

### Objective
File names lie. Hashes do not. What is the SHA256 of the payload file?

Format: 64-character hex string

### Evidence
The SHA256 hash of the payload file Sarah_Chen_Notes.exe is 224462ce5e3304e3fd0875eeabc829810a894911e3d4091d4e60e67a2687e695. The hash is consistent across both recorded instances of the file — the original location in C:\Users\vmAdminUsername\Documents\PHTG\ on 12/12 and the copy moved to C:\ProgramData\PHTG\HealthCloud\ on 12/13 — confirming it is the same file regardless of location or name changes. This hash was recovered from the SHA256 field in the FileRenamed events identified in Q25.

<img width="885" height="667" alt="image" src="https://github.com/user-attachments/assets/246e49dd-8884-4cae-8cd0-eba946284c59" />


### Answer
224462ce5e3304e3fd0875eeabc829810a894911e3d4091d4e60e67a2687e695
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 28: <Technique Name></strong></summary>  
### Q28 - Final File Name

### Objective
Using the SHA256 from Q27, track the file forward through rename events. What is the final observed file name tied to this hash?

Format: FileName

### Evidence
Tracking the payload hash 224462ce5e3304e3fd0875eeabc829810a894911e3d4091d4e60e67a2687e695 forward through the rename chain identified in Q25, the final observed filename was PHTG.exe. On 2025-12-13 at 10:16:22 UTC, the file was renamed from Sarah_Chen_Notes.exe to PHTG.exe in C:\ProgramData\PHTG\HealthCloud\ — deliberately named to blend in with PHTG's legitimate HealthCloud service directory, making it harder to identify as malicious during a casual review of running processes or installed files.
<img width="961" height="697" alt="image" src="https://github.com/user-attachments/assets/9adcfe67-6cb4-449f-8760-617f9fa32913" />

12/12 14:11 — Downloaded as Sarah_Chen_Notes.Txt.crdownload to C:\Users\vmAdminUsername\Downloads\ — the .crdownload extension indicates it was downloaded via Chrome/Edge
12/12 14:13 — Download completed, renamed to Sarah_Chen_Notes.Txt
12/12 14:13 — Moved to C:\Users\vmAdminUsername\Documents\PHTG\
12/12 14:14 — Renamed to Sarah_Chen_Notes.exe.Txt — double-extension applied
12/12 14:18 — Renamed to Sarah_Chen_Notes.exe — converted to executable
12/13 10:14 — Copied to C:\ProgramData\PHTG\HealthCloud\
12/13 10:16 — Renamed to PHTG.exe — final form, blending into the legitimate HealthCloud directory

### Answer
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 29: <Technique Name></strong></summary>  
### Q29 - File Classification

### Objective
External reputation services may not know this sample. MDE's own detection engine does. According to Microsoft Defender telemetry on the device, what software family is this file classified under?

Format: Malware family name
### Evidence
Windows Defender classified the payload as Trojan:Win32/Meterpreter.RPZ!MTB, identifying the malware family as Meterpreter — a well-known post-exploitation framework payload commonly deployed via Metasploit.
<img width="930" height="671" alt="image" src="https://github.com/user-attachments/assets/5b348c37-3111-4c24-b2e6-b36357ec2353" />

### Answer
Meterpreter
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 30: <Technique Name></strong></summary>  
### Q30 - Why Did It Run?

### Objective
Defender quarantined the payload three times between 14:11 and 14:17 on 12 December. Minutes later the same file executed without being blocked. What change to Defender's operating state allowed execution to proceed?

Format: Defender operating mode

### Evidence

The AdditionalFields JSON from the AntivirusDetectionActionType events in Q29 explicitly states "ReportSource":"Windows Defender Antivirus passive mode". Defender was quarantining the payload during its initial download and rename stages, but was subsequently switched to passive mode — a state where Defender detects and reports threats but does not block or remediate them.

<img width="938" height="693" alt="image" src="https://github.com/user-attachments/assets/d4f9d215-b1c2-4330-8d45-053fc8505823" />

### Answer
Passive Mode
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 31: <Technique Name></strong></summary>  
### Q31 - First Execution

### Objective
The payload executed multiple times across two phases. What filename did it run under during the first phase?

Format: FileName
### Evidence
So the payload first ran as Sarah_Chen_Notes.exe — three times on 12/12 at 14:18, 14:20, and 14:22 UTC. Each time it was launched straight from explorer.exe under vmadminusername, so the attacker was just double-clicking it from the file explorer like a normal user would.

Then on 12/13 it shifts into the second phase — now it's running as PHTG.exe, and this time it's not being manually clicked. It's being launched by cmd.exe executing Launch.bat from inside the HealthCloud directory.

<img width="936" height="720" alt="image" src="https://github.com/user-attachments/assets/dfb39509-d094-4c2d-aab4-1c8ec8a46347" />


### Answer
Sarah_Chen_Notes.exe
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 32: <Technique Name></strong></summary>  
### Q32 - Parent Process

### Objective
The payload's later executions were launched differently to the initial ones. Which process initiated the later phase?

Format: InitiatingProcessFileName

### Evidence
The later execution phase on 12/13 was no longer being manually launched from explorer.exe like the first phase. Instead, cmd.exe was the initiating process, running cmd.exe /c ""C:\ProgramData\PHTG\HealthCloud\Launch.bat"" — meaning the attacker had wired up Launch.bat to handle execution automatically. That's the shift from manual interaction to an automated persistence mechanism. We already had this from the DeviceProcessEvents results pulled in Q31.

<img width="927" height="713" alt="image" src="https://github.com/user-attachments/assets/08eb5929-18d7-478e-ae40-34e92520e6c8" />

### Answer
cmd.exe
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 33: <Technique Name></strong></summary>  
### Q33 - Batch File Wrapper

### Objective
The later executions were not launched directly. A command shell wrapped them. From the initiating command line, what is the full path of the .bat file that ran the payload?

Format: Full file path

### Evidence
The batch file responsible for wrapping the payload was C:\ProgramData\PHTG\HealthCloud\Launch.bat. Looking at the DeviceProcessEvents results, we can see PHTG.exe running twice — at 10:21:48 and 10:22:36 UTC on 12/13 — both times kicked off by cmd.exe running the same command: cmd.exe /c ""C:\ProgramData\PHTG\HealthCloud\Launch.bat".

<img width="935" height="647" alt="image" src="https://github.com/user-attachments/assets/f7ceae36-0f30-4a11-b815-9fa8207d5802" />

### Answer
C:\ProgramData\PHTG\HealthCloud\Launch.bat
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 34: <Technique Name></strong></summary>  
### Q34 - C2 IP

### Objective
After execution, what external IP did the compromised device attempt to communicate with?

Format: IPv4 address

### Evidence
After the payload ran, it tried calling back to 173.244.55.130 on port 4444 — that's the default Meterpreter C2 port, so this is exactly what you'd expect to see. It tried three times across both execution phases — twice as sarah_chen_notes.exe (12/12 and 12/13) and once as phtg.exe (12/13). All three came back as ConnectionFailed, so the C2 server wasn't home at the time — but the intent is pretty obvious.
What makes this even more interesting is that 173.244.55.130 sits in the exact same /24 subnet as the Uruguay RDP IPs we found earlier (173.244.55.128 and 173.244.55.131). Same infrastructure block being used for both the RDP brute-force and the Meterpreter C2 — that's the same threat actor across the whole chain, from initial access all the way through to payload execution.

https://www.rapid7.com/blog/post/2013/02/14/port-forwarding-how-to-verify-that-the-payload-can-connect-back-to-metasploit/

<img width="927" height="713" alt="image" src="https://github.com/user-attachments/assets/1df5a279-b0ed-44a8-9edc-d93b0d7600f9" />

### Answer
173.244.55.130
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 35: <Technique Name></strong></summary>  
### Q35 - C2 Geography

### Objective
Where is the C2 infrastructure located?

Format: Country, Continent
### Evidence
The C2 infrastructure at 173.244.55.130 is located in Uruguay, South America. This confirms what was already suspected — the same threat actor used infrastructure in Uruguay for the RDP brute-force (173.244.55.128 and 173.244.55.131), the successful logons, and now the Meterpreter C2 server.

<img width="933" height="428" alt="image" src="https://github.com/user-attachments/assets/cb6afc13-6ca4-41b7-9c9c-061248d6ce64" />

### Answer
Uruguay, South America
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 36: <Technique Name></strong></summary>  
### Q36 - C2 Remote Port

### Objective
Threat actors often avoid 80/443 after execution. What remote port did the post-execution process use?

Format: Port number

### Evidence
The payload consistently called back to the C2 server on port 4444 across all three connection attempts. This is the default Meterpreter reverse TCP port and a deliberate choice — rather than blending in with normal web traffic on 80/443, the attacker used 4444, which is only significant if there's nothing on the network monitoring for it

https://www.rapid7.com/blog/post/2013/02/14/port-forwarding-how-to-verify-that-the-payload-can-connect-back-to-metasploit/

<img width="929" height="486" alt="image" src="https://github.com/user-attachments/assets/d69b2c57-7e68-4453-ae79-8e709d07beb3" />

### Answer
4444
</details> 

<details>
<summary id="-flag-2">🚩 <strong>Flag 37: <Technique Name></strong></summary>  
### Q37 - Repurposed Baseline

### Objective
The attacker did not write persistence from scratch. They placed the final payload inside a directory belonging to a legitimate internal service rolled out the same week. What was the name of that service?

Format: Service or directory name

### Evidence
The attacker didn't build their own persistence directory — they just moved into one that already existed. The final payload PHTG.exe and the batch file Launch.bat were both dropped into C:\ProgramData\PHTG\HealthCloud\, which is the legitimate HealthCloud service directory rolled out just the day before on 11 December 2025. Hiding malicious files inside a brand new internal service is smart — anyone glancing at that folder would assume everything in it belongs there. I spotted this across the file rename and execution events tracked from Q25 through Q33.
 
<img width="921" height="725" alt="image" src="https://github.com/user-attachments/assets/475ef1da-cf69-4e90-a8f8-3e325eb2d70f" />

### Answer
HealthCloud
</details> 



## Mitigation Recommendations

* No RDP exposure controls: The VM azwks-phtg-02 was deployed with a public IP and RDP open to the internet with no NSG restriction. 173 unique IPs established accepted connections on port 3389, and 23 successful authentications originated from Uruguay — a country entirely outside PHTG's operating region. A basic NSG rule restricting RDP to known IP ranges or a VPN requirement would have prevented the entire attack chain.
* Weak credential on an internet-facing VM: The account vmadminusername — a generic, predictable administrator account name — was the only thing standing between the open internet and the VM. 646 brute-force attempts were made before 23 successful logons were recorded. A strong, non-guessable account name combined with an account lockout policy would have significantly raised the barrier to entry.
* Defender running in passive mode: Windows Defender detected and flagged Trojan:Win32/Meterpreter.RPZ!MTB three times but took no remediation action because it was operating in passive mode. The payload executed freely after each detection. Defender should have been in active mode to quarantine the file on first detection.
* No OPSEC controls on social media: The LinkedIn post by Sarah Chen exposed the VM name, public IP, subscription ID, and Azure portal details to the open internet. The first scanning activity hit port 3389 within hours of the post going live. A clear OPSEC policy prohibiting the sharing of infrastructure details on social media would have prevented the initial exposure entirely.

## Remediation Actions

* Immediately restrict RDP access on azwks-phtg-02 via NSG rules — RDP should never be exposed directly to the internet.
* Disable or rename the vmadminusername account, reset credentials, and enforce a strong password policy across all VM administrator accounts.
* Switch Windows Defender from passive mode to active mode and verify protection status across all endpoints in the environment.
* Remove PHTG.exe, Launch.bat, and all associated payload files from C:\ProgramData\PHTG\HealthCloud\ and audit the directory for any additional attacker-placed files.
* Block 173.244.55.128, 173.244.55.130, and 173.244.55.131 at the network perimeter — these IPs were used for both RDP brute-force and Meterpreter C2.
* Review and audit all files opened during the compromised sessions, particularly notes_sarah.txt, to assess what internal information the attacker may have accessed.
* Establish a formal OPSEC policy governing what infrastructure information employees may share on social media and professional networking platforms.

