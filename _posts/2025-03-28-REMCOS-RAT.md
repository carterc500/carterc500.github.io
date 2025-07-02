---
layout: post
title: REMCOS RAT
subtitle: REMCOS RAT Writeup
tags: [threat-intel]
comments: true
mathjax: true
---

## Executive Summary
 

## Key Takeaways  
This is an anlysis of an instance of the REMCOS RAT that was seen in late March of 2025. Access to the RAT (Remote Access Trojan) was obtained through [MalwareBazaar](https://bazaar.abuse.ch/) by downloading a sample from the database. Static analysis was used in determing how the malware worked to eventually download the payload. In this case dynamic analysis was not determined to be needed as the original sample was a JavaScript file that made it easy to analyze. 
  --TODO-- File hash

This is intended to be mainly for the purposes of documenting an instance of malware and so this can be used by anyone interested in this specific instance of REMCOS or just by anyone else interested. 
  
In terms of the victim of this attack, that is unknown as the sample was downloaded from a public database which does not contain context of who provided the sample. Due to this lack of information, it also makes it harder to tell who the possible attacker could be as motivation can't be inferred.  
--TODO-- Possible attestation?  
  
The main result of this analysis was improvement of my own skills in malware analysis and getting experience with multiple different attack techniques. Additionally this analysis shows the importance of restricting user access to least priviledge, not trusting even generally "safe" file types such as images, and blocking access (specifically download access) to unknown IPs/destinations. 
## Assessment

## Indicators of Compromise


Main indicator of compromise is successful communication with httx://104[.]168[.]7[.]32 where the new_image.jpg is downloaded. Further evidence 
--TODO-- paste[.]ee and file hash

#### Endpoint Artifacts
- Original JS file - SHA256 971ae4e4aa24029751d0c76ece96dec196d05b47e8fb218aa87ecd3ccd8513dff  
- new_image.jpg - SHA256 cd5d382270649f01f85f42beaff153497af774ae6e623407c56e694621323fc2  
--TODO-- Copy over VirusTotal report https://www.virustotal.com/gui/file/cd5d382270649f01f85f42beaff153497af774ae6e623407c56e694621323fc2/behavior  

| Artifcat | Type | Description | Tactic |
| :------ |:--- | :--- | :--- |
| Five | Six | Four | sixty |
| Ten | Eleven | Nine | 
| Seven | Eight | Six |
| Two | Three | One |


#### Network Artifacts
- hxxp://104[.]168[.]7[.]32  
- hxxp://paste.ee/d/svCzpzA6 (23[.]186[.]113[.]60)  

#### Malware

#### Common Vulnerabilities & Exposures (CVEs)

## MITRE ATT&CK Techniques

## Detection Oportunities

## Appendices