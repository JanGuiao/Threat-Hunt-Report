
# Threat Hunt Report: Scattered Invoice (BEC Investigation)
#### Belated participant: Jan Guiao
#### Date: April 2026


## Platforms and Languages Used

### Platforms/Workspace:
* Microsoft Sentinel
* Logs Analytics Workspace (law-cyber-range)

### Languages/Tools:
* Kusto Query Language (KQL) for querying device events, registry search, and persistence.


## General:
A bank just froze a £24,500 wire transfer.
Finance says they followed procedure. They got an email from a colleague with updated banking details and processed it. That colleague is Mark Smith.
Mark reported weird MFA notifications last night. He approved one just to make them stop.
This morning his team found inbox rules nobody created.
You're the analyst. Find out what happened.

## Whats to Investigate?
* MFA fatigue attacks
* Inbox rule manipulation
* Business email compromise
* Cloud app lateral access
* Session correlation across Entra ID and Defender

## Flag Analysis
<details>
<summary id="-flag-1">🚩 <strong>Flag 1: <Technique Name></strong></summary>  

### Q01 - Compromised Account 

### Objective 
IR Lead: "We have a confirmed BEC. £24,500 redirected. Finance say they got an email from Mark Smith with updated banking details. Mark reported weird MFA notifications last night. He approved one to make them stop. I need you in the sign-in logs. Confirm the compromise, find the attacker's infrastructure. Clock is running."

Before you can investigate, confirm the compromised identity. Query SigninLogs for the user identified in the incident.

<img width="833" height="487" alt="image" src="https://github.com/user-attachments/assets/e229853d-1c22-4173-afe0-2b7fcce12aae" />

### Answer: m.smith@lognpacfic.org
</details> 


<details>
<summary id="-flag-2">🚩 <strong>Flag 2: <Technique Name></strong></summary>  

## Q02 - Attacker Source IP

### Objective 
Mark authenticated from his usual location during the day. But someone else authenticated as Mark from somewhere else during the evening. Two IPs, one account.

Baseline Mark's sign-in activity. Compare successful authentications before the attack window against the evening activity. Geography and timing will separate legitimate from malicious.

Format: IPv4 address

### Evidence: 
We actually already have the answer from the Q01 results — two IPs showed up in Mark's sign-in data:

* 172.175.65.103 — US — one login, during the early part of the window — Mark's legitimate IP
* 205.147.16.190 — NL (Netherlands) — repeated sessions throughout the evening — the attacker

<img width="895" height="392" alt="image" src="https://github.com/user-attachments/assets/eed4f246-2754-455f-a251-6e5e9f56005d" />

### Answer: 205.14.16.190

</details> 

<details>
<summary id="-flag-3">🚩 <strong>Flag 3: <Technique Name></strong></summary>  

### Q03 - Attack Origin Country

### Objective 
You have the attacker's IP. Now determine where it geolocates to. Does an employee in one country suddenly authenticating from another make sense?

Format: Two-letter country code (e.g., US, GB, DE)

### Evidence: 
Already in our Q02 results — the attacker IP 205.147.16.190 was shown NL in the Location column.


<img width="901" height="352" alt="image" src="https://github.com/user-attachments/assets/f372808a-85a3-4f88-8294-1e2a6ef469d5" />


### Answer: NL
</details> 

<details>
<summary id="-flag-4">🚩 <strong>Flag 4: <Technique Name></strong></summary>  

### Q04 - Compromised Account 

### Objective 
What error code indicates that strong authentication (MFA) was required but not completed?

Format: Numeric error code

### Evidence: 
"Strong Authentication is required" — hit twice by the attacker from 205.147.16.190 (NL) at 21:54 and 22:24 before Mark approved the MFA fatigue push and let them through.

<img width="895" height="548" alt="image" src="https://github.com/user-attachments/assets/4d53967a-02a4-4dd4-95a9-83b33c082a3a" />


### Answer: 50074
</details> 

<details>
<summary id="-flag-5">🚩 <strong>Flag 5: <Technique Name></strong></summary>  

### Q05 - Compromised Account 

### Objective 
Q05 - MFA Fatigue Intensity
75
Mark said he was getting repeated MFA push notifications on his phone. He denied them but eventually approved one just to make them stop.

How many MFA push requests did Mark deny before he approved one?

Format: Number

### Evidence: 
The platform is counting 50140 as part of the MFA fatigue sequence, which makes sense in context — those interrupts are also part of the attacker being blocked before Mark approved. So 2 + 1 = 3 when combining the two distinct 50140 descriptions plus the 50074 denials isn't quite it either.
Most likely the platform is counting 2 x 50074 + 1 x 50140 = 3 total blocks before the first successful.

<img width="917" height="428" alt="image" src="https://github.com/user-attachments/assets/e47cecbb-fcaf-4a3e-82c2-5f6e2ee15642" />


### Answer: 3

</details> 

<details>
<summary id="-flag-6">🚩 <strong>Flag 6: <Technique Name></strong></summary>  

### Q06 - Application Accessed

### Objective 
After beating MFA, the attacker accessed a specific Microsoft application. A remote attacker without the desktop app installed would use the web version. What application did the attacker authenticate to?

Format: Application name as logged

### Evidence: 
The format instruction says "Application name as logged" — so the answer is pulled directly from the AppDisplayName field in SigninLogs, which logged it as One Outlook Web.

<img width="899" height="678" alt="image" src="https://github.com/user-attachments/assets/3acbbab2-c8e6-49f1-8ef8-5df6f4ac453a" />

### Answer: One Outlook Web

</details> 

<details>
<summary id="-flag-7">🚩 <strong>Flag 7: <Technique Name></strong></summary>  

### Q07 - Compromised Account 

### Objective 
Mark's corporate device runs Windows on a managed endpoint. The attacker authenticated from something very different. Comparing device profiles between legitimate and malicious sessions is how you spot compromised accounts at scale.

Format: Operating system name

### Evidence:
In the previous screenshot, within the "User Agent" column, the User Agent string contains X11; Ubuntu; Linux x86_64 — X11 is the display system, Linux is the OS.

<img width="899" height="678" alt="image" src="https://github.com/user-attachments/assets/181f71f9-1adc-44e8-a0b6-8a2c548e3637" />

### Answer: Linux

</details> 

<details>
<summary id="-flag-8">🚩 <strong>Flag 8: <Technique Name></strong></summary>  

### Q08 - Attacker Browser

### Objective 
Cross-reference with Mark's normal browser. Different browser, different OS, different country. Three anomaly layers beyond just the IP.

Format: Browser name and version as logged

### Evidence: 
The User Agent format always ends with BrowserName/Version — so Firefox/147.0 is exactly how it was logged.

<img width="895" height="691" alt="image" src="https://github.com/user-attachments/assets/f4349165-c94d-4a99-91d6-ac4bd8eabfaa" />

### Answer: Firefox 147.0

</details> 

<details>
<summary id="-flag-9">🚩 <strong>Flag 9: <Technique Name></strong></summary>  

### Q09 - First Post-Auth Action

### Objective 
The first action after authentication reveals intent. Did they exfiltrate immediately? Set up persistence? Or read the inbox to understand the target?

Query CloudAppEvents for the attacker's IP during the attack window. Sort by time ascending. What was the very first ActionType?

Format: ActionType value as logged

### Evidence: 
This action type is significant because it is triggered by the OAuth token the attacker obtained after beating MFA — at 21:56, roughly 3 minutes before their first clean ResultType == 0 signin at 21:59 in SigninLogs.

<img width="905" height="480" alt="image" src="https://github.com/user-attachments/assets/4a8233b3-d417-473a-afe9-04b4b652a477" />

### Answer: MailItemsAccessed

</details> 

<details>
<summary id="-flag-10">🚩 <strong>Flag 10: <Technique Name></strong></summary>  

### Q10 - Rule Creation Method

### Objective
Sophisticated attackers establish persistence to maintain access. Inbox rules are a favourite. They are silent, persistent, and often overlooked.

Query CloudAppEvents for the attack timeframe. Look at the ActionType field for anything related to email rule creation.

Format: ActionType value as logged

### Evidence: 
Filtered CloudAppEvents for the attacker's IP and used ActionType contains "Rule" to isolate rule-related activity — returned two New-InboxRule events from 205.147.16.190 at 22:02 and 22:03 via Exchange Online.

<img width="893" height="388" alt="image" src="https://github.com/user-attachments/assets/6879e12b-a925-4837-bdb6-fc1a6020b50c" />


### Answer: New-InboxRule

</details> 

<details>
<summary id="-flag-11">🚩 <strong>Flag 11: <Technique Name></strong></summary>  

### Q11 - Forward Rule Name

### Objective
Attackers name rules strategically to be as inconspicuous as possible. Examine the RawEventData for the inbox rule creation event. Find the Name parameter.

Format: Exact rule name (case-sensitive, may be a single character)

### Evidence: 
Two New-InboxRule events created 90 seconds apart from 205.147.16.190. Rule 1 named . forwards finance-keyword emails to insights@duck.com. Rule 2 named .. silently deletes incoming security alerts — cover track to prevent Mark seeing any compromise notifications.

<img width="691" height="399" alt="image" src="https://github.com/user-attachments/assets/6e2a19a8-0d7d-4092-b76a-abb83b1179e3" />


### Answer: .

</details> 

<details>
<summary id="-flag-12">🚩 <strong>Flag 12: <Technique Name></strong></summary>  

### Q12 - Forward Destination

### Objective 
The external email receiving forwarded messages is attacker-controlled infrastructure. This is a critical IOC for email gateway blocking.

Find the ForwardTo parameter in the rule configuration.

Format: email@domain.tld

### Evidence: 
{"Name":"ForwardTo","Value":"insights@duck.com"} — any email hitting Mark's inbox containing invoice, payment, wire, transfer gets silently forwarded to this address.

<img width="691" height="399" alt="image" src="https://github.com/user-attachments/assets/2b8c6dbf-88df-4e0a-9f24-cdf6999a0f11" />


### Answer: insights@duck.com

</details> 

<details>
<summary id="-flag-13">🚩 <strong>Flag 13: <Technique Name></strong></summary>  

### Q13 - Forward Keywords

### Objective
The keywords triggering the forwarding rule reveal what data the attacker wants. Financial keywords indicate invoice fraud.

Find the SubjectOrBodyContainsWords parameter.

Format: Keywords as logged, comma-separated

### Evidence: 
SubjectOrBodyContainsWords parameter from the forwarding rule — any email hitting those keywords gets silently forwarded to insights@duck.com. As shown in the picture these keywords are logged, comma-separated.

<img width="907" height="423" alt="image" src="https://github.com/user-attachments/assets/c1fcf523-b5fb-4336-ab8f-a4affe5906f1" />


### Answer: invoice, payment, wire, transfer

</details> 

<details>
<summary id="-flag-14">🚩 <strong>Flag 14: <Technique Name></strong></summary>  

### Q14 - Rule Processing Flag 

### Objective
Smart attackers configure rules so no other rules process the matched emails after theirs. This prevents the user's own rules from alerting them.

What parameter ensures this rule takes priority over all others?

Format: Parameter name as logged

### Evidence:
Set to True in both rules — once a matching email hits the attacker's rule, no subsequent inbox rules are evaluated. Prevents any of Mark's legitimate rules from flagging, moving, or alerting on the same email.

<img width="908" height="479" alt="image" src="https://github.com/user-attachments/assets/ad24a9ed-dbc8-4834-b77f-1b7d0799e505" />

### Answer: StopProcessingRules

</details> 

<details>
<summary id="-flag-15">🚩 <strong>Flag 15: <Technique Name></strong></summary>  

### Q15 - Delete Rule Name

### Objective 
Inbox rules can delete messages as easily as forward them. Attackers create rules to automatically delete security notifications and suspicious activity alerts.

Query CloudAppEvents for ALL inbox rule creation events during the attack window. What is the second rule's name?

Format: Exact rule name (case-sensitive, may be a single character)

### Evidence: 
The first rule we saw was named value with "." in which the action was "ForwardTo - insights@duck.com"; invoice, payment, wire, transfer. A second rule was made named value ".." in which the action was "DeleteMessage: True"; suspicious, security, phishing, unusual, compromised, verify.
<img width="896" height="689" alt="image" src="https://github.com/user-attachments/assets/d1bd7b78-eff1-4032-b5d0-de7e10519955" />

### Answer: ..

</details> 

<details>
<summary id="-flag-16">🚩 <strong>Flag 16: <Technique Name></strong></summary>  

### Q16 - Delete Keywords

### Objective 
The keywords targeted for deletion reveal what the attacker wants to hide. Security-related terms mean they are hiding breach notifications.

Parse the second rule's configuration. Find the keyword triggers.

Format: Keywords as logged, comma-separated
### Evidence: 
SubjectOrBodyContainsWords from the ".." delete rule signified the previous screenshot in Q15; suspicious, security, phishing, unusual, compromised, verify.
<img width="896" height="689" alt="image" src="https://github.com/user-attachments/assets/d1bd7b78-eff1-4032-b5d0-de7e10519955" />

### Answer: suspicious, security, phishing, unusual, compromised, verify

</details> 

<details>
<summary id="-flag-17">🚩 <strong>Flag 17: <Technique Name></strong></summary>  

### Q17 - BEC Target

### Objective 
Pivot to EmailEvents. Filter by the compromised account as sender and the attacker's IP. Who received the fraudulent email?

Format: email@domain.tld

### Evidence: 
The RecipientEmailAddress field in the result shows j.reynolds@lognpacific.org — that's the address the attacker sent the fraudulent email to from Mark's session. One email, one recipient, sent directly from the attacker's IP 205.147.16.190 at 22:06.

<img width="909" height="636" alt="image" src="https://github.com/user-attachments/assets/d7187d84-e43b-48a5-afbe-fe2e43a04c31" />


### Answer: j.reynolds@lognpacific.org

</details> 

<details>
<summary id="-flag-18">🚩 <strong>Flag 18: <Technique Name></strong></summary>  

### Q18 - BEC Subject Line

### Objective 
The subject line reveals the social engineering pretext. The attacker replied to an existing thread rather than creating a new email. Classic thread hijacking.

Format: Full subject line as logged

### Evidence: 
In the previous query Q17- The subject column highlights the subject line of the invoice "RE: Invoice #INV-2026-0892 - Updated Banking Details"
<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/0e27d1c1-64f2-4f62-b9c3-fd6532318cbb" />


### Answer: RE: Invoice #INV-2026-0892 - Updated Banking Details

</details> 

<details>
<summary id="-flag-19">🚩 <strong>Flag 19: <Technique Name></strong></summary>  

### Q19 - Email Direction

### Objective 
Was this email sent externally or within the organisation? The direction determines whether email gateway rules could have caught it.

Format: EmailDirection value as logged

### Evidence: 
The fraudulent email was routed internally between two employees on the same domain, classified as Intra-org in EmailDirection — meaning it bypassed external email gateway controls entirely.

<img width="891" height="513" alt="image" src="https://github.com/user-attachments/assets/1f1d87fb-42bf-430e-81f7-7ee8a85eb3ed" />


### Answer: Intra-org

</details> 

<details>
<summary id="-flag-20">🚩 <strong>Flag 20: <Technique Name></strong></summary>  

### Q20 - BEC Sender IP

### Objective 
Cross-correlate. The SenderIPv4 on the BEC email should match the attacker's sign-in IP. This proves the same session was used for authentication and email sending.

Format: IPv4 address

### Evidence: 
SenderIPv4 on the fraudulent email matches the attacker's authenticated session IP from SigninLogs exactly — same IP used to beat MFA, create inbox rules, and send the BEC email. 

<img width="898" height="513" alt="image" src="https://github.com/user-attachments/assets/199d7fa3-35a9-4a25-b3cd-929c3e7a3470" />


### Answer: 205.147.16.190

</details> 

<details>
<summary id="-flag-21">🚩 <strong>Flag 21: <Technique Name></strong></summary>  

### Q21 - Cloud App Accessed

### Objective 
Query CloudAppEvents filtered to the attacker's IP. Look for file access ActionTypes. Check the Application field.

Format: Application name as logged

### Evidence: 
Beyond email, the attacker accessed "Microsoft OneDrive for Business" — FileAccessed, ListViewed, ListCreated, and Broke sharing inheritance all logged against the attacker's IP. The Broke sharing inheritance action is particularly significant as it suggests they may have modified file permissions to enable external access to documents.
<img width="895" height="688" alt="image" src="https://github.com/user-attachments/assets/7bfcd2a6-301f-4ec5-9f3c-bd175602cf65" />
<img width="886" height="677" alt="image" src="https://github.com/user-attachments/assets/24ffce61-47eb-4759-866b-a81f2b8729a9" />

### Answer: Microsoft OneDrive for Business

</details> 

<details>
<summary id="-flag-22">🚩 <strong>Flag 22: <Technique Name></strong></summary>  

### Q22 - SharePoint App Accessed

### Objective 
The attacker did not stop at personal files. The sign-in logs show authentication to another Microsoft cloud application during the attack window.

Query SigninLogs for the attacker's IP. What other application did they access?

Format: Application display name (e.g. Microsoft OneDrive for Business)

### Evidence: 
The query filtered CloudAppEvents to the attacker's IP 205.147.16.190 across the investigation window, then used summarize count() by Application to group all activity by application name. This produced a distinct list of every cloud app the attacker touched during their session — Microsoft Exchange Online, Microsoft OneDrive for Business, and Microsoft SharePoint Online all surfaced as logged application names. SharePoint was the answer because it was the second app beyond Exchange that the attacker accessed, and the Application field in CloudAppEvents logged it as Microsoft SharePoint Online exactly matching the platform's expected format.

<img width="893" height="574" alt="image" src="https://github.com/user-attachments/assets/a01328d0-981a-4dab-8e97-ea6578622aba" />

### Answer: Microsoft SharePoint Online

</details> 

<details>
<summary id="-flag-23">🚩 <strong>Flag 23: <Technique Name></strong></summary>  

### Q23 - Session Correlation

### Objective 
One identifier links all attacker activity across sign-ins, inbox rules, and email. Find the session ID that ties the entire investigation together.

Check the CloudAppEvents inbox rule events. In RawEventData, find AppAccessContext.AADSessionId. Then confirm it matches the SessionId in SigninLogs for the attacker's successful authentication.

Format: GUID (xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)

### Evidence: 
The attacker's entire intrusion — MFA bypass, inbox rules, OneDrive and SharePoint access, and the fraudulent email — all executed under a single AAD session ID, confirmed consistent across both SigninLogs and CloudAppEvents.

<img width="885" height="695" alt="image" src="https://github.com/user-attachments/assets/8e1c25ad-cad6-41ae-9e8c-37b48589eb01" />


### Answer: 00225cfa-a0ff-fb46-a079-5d152fcdf72a


</details> 

<details>
<summary id="-flag-24">🚩 <strong>Flag 24: <Technique Name></strong></summary>  

### Q24 - Conditional Access Status

### Objective 
Conditional Access policies can block sign-ins from unmanaged devices or risky locations. Check the attacker's successful sign-in. What was the ConditionalAccessStatus?

Format: Status value as logged

### Evidence: 
No Conditional Access policy was evaluated against the attacker's session — meaning there were no policies configured to block or challenge sign-ins from foreign IPs, unmanaged devices, or risky locations.
<img width="898" height="461" alt="image" src="https://github.com/user-attachments/assets/a4f3da4d-2bb7-44ee-b955-a00055722103" />


### Answer: notApplied

</details> 

<details>
<summary id="-flag-25">🚩 <strong>Flag 25: <Technique Name></strong></summary>  

### Q25 - MFA Fatigue MITRE ID

### Objective 
Map the MFA fatigue technique to the MITRE ATT&CK framework. What technique ID describes repeated MFA push notifications to wear down the user?

Format: MITRE technique ID (e.g., T1234)

### Evidence: 
As per https://attack.mitre.org/techniques/T1621/, MITRE ATT&CK T1621 is a docuimented technique in the framework under the Credential Access tactic, added specifically to capture MFA fatigue attacks after Scattered Spider and similar threat actors popularised it. 

### Answer: T1621

</details> 

<details>
<summary id="-flag-26">🚩 <strong>Flag 26: <Technique Name></strong></summary>  

### Q26 - Email Rules MITRE ID

### Objective 
The attacker created inbox rules to hide evidence. Map this defence evasion technique to MITRE ATT&CK. What technique ID specifically describes hiding malicious activity through email rules?

Format: MITRE technique ID with sub-technique (e.g., T1234.001) 

### Evidence: 
As per https://attack.mitre.org/techniques/T1564/008/, specifically covers inbox rules created to forward, delete, or otherwise suppress emails to conceal attacker activity, exactly matching the . and .. rules created in Mark's mailbox.

### Answer: T1564.008

</details> 

<details>
<summary id="-flag-27">🚩 <strong>Flag 27: <Technique Name></strong></summary>  

### Q27 - Credential Source

### Objective 
The threat group behind this attack is known for purchasing credentials from a specific source. These credentials are harvested by malware that steals saved passwords, session tokens, and browser data from infected machines.

What type of malware typically provides initial credentials to groups like this?

Format: Malware category (one word)
### Evidence: 

Scattered Spider (also known as UNC3944) is known for purchasing stolen credentials from infostealer logs sold on Dark Web marketplaces.  Infostealers  harvest saved browser passwords, session cookies, and autofill data from infected machines — giving attackers valid credentials before they ever attempt login, explaining why they went straight to MFA fatigue rather than brute force.

### Answer: Infostealer

</details> 

<details>
<summary id="-flag-28">🚩 <strong>Flag 28: <Technique Name></strong></summary>  

### Q28 - Immediate Containment

### Objective
The attacker still has a valid session. The inbox rules are still active. What is the first containment action?

Format: Two-word action

### Evidence: 
Invalidates the AAD session token 00225cfa-a0ff-fb46-a079-5d152fcdf72a immediately — kicks the attacker out of every active connection across Outlook, OneDrive, and SharePoint in one action. Everything else — removing inbox rules, resetting the password, blocking the IP — comes after, but none of it matters while the attacker still holds a valid session token.

## Importance
The attacker's session token is still valid regardless of anything else you do. Even if you reset Mark's password right now, the attacker stays connected — password resets don't invalidate active sessions in Azure AD. Revoking the session forces re-authentication on every active connection, instantly cutting the attacker off from Outlook, OneDrive, and SharePoint simultaneously. It's the only action that stops the bleeding in real time.

### Answer: Revoke Session

</details> 

<details>
<summary id="-flag-29">🚩 <strong>Flag 29: <Technique Name></strong></summary>  

### Q29 - Threat Actor Attribution

### Objective 
Throughout this investigation you observed MFA fatigue, inbox rule persistence, BEC targeting finance, and use of anonymising infrastructure. The briefing mentioned a group that targeted MGM Resorts and Caesars Entertainment.

Who did this?

Format: Threat group name (two words)
### Evidence: 
TTPs across this investigation are textbook Scattered Spider: MFA fatigue via push bombing, session token abuse, inbox rule persistence with minimal-character names, BEC targeting finance, and routing through anonymising infrastructure in the Netherlands. Tactics such as: 
* MFA fatigue - their signature initial access method, documented across MGM, Caesars, and multiple BEC cases
* Inbox rules with minimal names - . and .. are deliberately inconspicuous, consistent with their documented persistence.
* Thread hijacking BEC -  replying to an existing invoice thread rather than creating a new email is a known Scattered Spider social engineering technique.
* (e.g.)

## Importance:
Scattered Spider is one of the most prolific financially motivated threat actors currently active, responsible for hundreds of millions in losses across MGM, Caesars, and numerous BEC campaigns. Attribution confirms the attack was targeted and sophisticated — not opportunistic — meaning other accounts and systems in the organisation should be treated as potentially compromised until ruled out.

### Answer: Scattered Spider

</details> 



## Last Statements
