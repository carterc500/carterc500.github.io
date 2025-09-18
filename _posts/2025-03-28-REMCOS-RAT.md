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
  
The main result of this analysis was improvement of my own skills in malware analysis and getting experience with multiple different attack techniques. Additionally this analysis shows the importance of restricting user access to least priviledge, not trusting even generally "safe" file types such as images, and blocking access (specifically download access) to unknown IPs/destinations. Also one big important lesson that came out of this is in the future when I do these types of excersises is I need to complete them quickly since the command and control servers can be taken down quickly and so I lose access to be able to get samples after.
## Assessment
--TODO-- Add in Raw Code image and talk more in depth about walking through the code and why the characters were replaced.  


Retrieved a sample of malware from [MalwareBazaar](https://bazaar.abuse.ch/) tagged as being a Remcos RAT (SHA256:971ae4e4aa24029751d0c76ece96dec196d05b47e8fb218aa87ecd3ccd8513dff). Analysis of the sample took place on a virtual machine running FlareVM.  
  
Began analysis by opening the JavaScript file with VisaulStudio code to start static analysis after it was verified that it was not an executable.  
![](/assets/img/file_type.png)  

On opening the file saw that there were recurring sets of non standard characters found throughout the file. Found and replaced each occurance of the random characters with an empty string until all that was left is seen below. The random set of characters can be seen on lines 2 and 3 highlighted in the yellow box.   
![](/assets/img/code_deobfuscated.png)  
With the code fully deobfuscated as seen in:  
![](/assets/img/true_code.png)  

Initially ran the URL through hxxp://paste[.]ee/d/svCzpzA6 [HybridAnalysis](https://hybrid-analysis.com/sample/0be185a53ab1c55478115df977406722afb37fa6e81dfbf81610d9acd32ce1f8) to see what it detected. As seen in the report the output was detected as malicious and so used a proxy service to retrieve the response from the URL. 
![](/assets/img/url_output.png)  

As seen once again the response is heavily obfuscated with this time instead of random characters being

## Indicators of Compromise

#### Endpoint Artifacts 

| Artifact | Type | Description | Tactic | Links |
| :------ |:--- | :--- | :--- |
| Original File | JS File/Downloader | SHA256 971ae4e4aa24029751d0c76ece96dec196d05b47e8fb218aa87ecd3ccd8513dff | [Initial Access](https://attack.mitre.org/tactics/TA0001/) | None |
| hxxp://paste[.]ee/d/svCzpzA6 Response | Download | Instructions for downloading new_image.jpg (SHA256 cd5d382270649f01f85f42beaff153497af774ae6e623407c56e694621323fc2) | [Execution](https://attack.mitre.org/tactics/TA0002/) | [VirusTotal](https://www.virustotal.com/gui/file/cd5d382270649f01f85f42beaff153497af774ae6e623407c56e694621323fc2/detection) |
| new_image.jpg | Malware | Remcos RAT downloaded | [Persistence](https://attack.mitre.org/tactics/TA0003/) | None |

#### Network Artifacts

| Artifact | Type | Description | Kill Chain Stage | Links |
| :------ |:----- | :--- | :--- |
| hxxp://paste[.]ee/d/svCzpzA6 | URL | URL contacted to retrieve malicious payload | Exploitation | [HybridAnalysis](https://hybrid-analysis.com/sample/0be185a53ab1c55478115df977406722afb37fa6e81dfbf81610d9acd32ce1f8) | 
| hxxp://104[.]168[.]7[.]32/xampp/sv/new_image.jpg | URL | Web server that was hosting the "new_image.jpg" file | Installation | [VirusTotal](https://www.virustotal.com/gui/url/faa25f04976015b3e97766ceedc652c2a89584c90aafc869f9d4ee923c280a88) |

#### Malware

| Malware | Hash Type | File Hash | Description | Malware Analysis Report | Kill Chain Stage |
| :------ |:----- | :--- | :--- | :--- | :----- |
|Remcos RAT | Unknown | Unkown | Would be downloaded as new_image.jpg but by time analysis was ongoing URL hosting the Malware was taken down and sample could not be retrieved | N/A | Command & Control |