
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
Already in our Q01 results — the attacker IP 205.147.16.190 consistently showed NL in the Location column.


<img width="901" height="352" alt="image" src="https://github.com/user-attachments/assets/f372808a-85a3-4f88-8294-1e2a6ef469d5" />


### Answer: NL
</details> 

<details>
<summary id="-flag-4">🚩 <strong>Flag 4: <Technique Name></strong></summary>  

### Q04 - Compromised Account 

### Objective 

### Evidence: 

### Answer:
</details> 

<details>
<summary id="-flag-5">🚩 <strong>Flag 5: <Technique Name></strong></summary>  

### Q05 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-6">🚩 <strong>Flag 6: <Technique Name></strong></summary>  

### Q06 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-7">🚩 <strong>Flag 7: <Technique Name></strong></summary>  

### Q07 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-8">🚩 <strong>Flag 8: <Technique Name></strong></summary>  

### Q08 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-9">🚩 <strong>Flag 9: <Technique Name></strong></summary>  

### Q09 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-10">🚩 <strong>Flag 10: <Technique Name></strong></summary>  

### Q10 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-11">🚩 <strong>Flag 11: <Technique Name></strong></summary>  

### Q11 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-12">🚩 <strong>Flag 12: <Technique Name></strong></summary>  

### Q12 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-13">🚩 <strong>Flag 13: <Technique Name></strong></summary>  

### Q13 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-14">🚩 <strong>Flag 14: <Technique Name></strong></summary>  

### Q14 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-15">🚩 <strong>Flag 15: <Technique Name></strong></summary>  

### Q15 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-16">🚩 <strong>Flag 16: <Technique Name></strong></summary>  

### Q16 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-17">🚩 <strong>Flag 17: <Technique Name></strong></summary>  

### Q17 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-18">🚩 <strong>Flag 18: <Technique Name></strong></summary>  

### Q18 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-19">🚩 <strong>Flag 19: <Technique Name></strong></summary>  

### Q19 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-20">🚩 <strong>Flag 20: <Technique Name></strong></summary>  

### Q20 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-21">🚩 <strong>Flag 21: <Technique Name></strong></summary>  

### Q21 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-22">🚩 <strong>Flag 22: <Technique Name></strong></summary>  

### Q22 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-23">🚩 <strong>Flag 23: <Technique Name></strong></summary>  

### Q23 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-24">🚩 <strong>Flag 24: <Technique Name></strong></summary>  

### Q24 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-25">🚩 <strong>Flag 25: <Technique Name></strong></summary>  

### Q25 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-26">🚩 <strong>Flag 26: <Technique Name></strong></summary>  

### Q26 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-27">🚩 <strong>Flag 27: <Technique Name></strong></summary>  

### Q27 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-28">🚩 <strong>Flag 28: <Technique Name></strong></summary>  

### Q28 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 

<details>
<summary id="-flag-29">🚩 <strong>Flag 29: <Technique Name></strong></summary>  

### Q29 - Compromised Account 

### Objective 

### Evidence: 

### Answer:

</details> 



## Last Statements
