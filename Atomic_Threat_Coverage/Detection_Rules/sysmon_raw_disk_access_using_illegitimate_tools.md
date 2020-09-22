| Title                    | Raw Disk Access Using Illegitimate Tools       |
|:-------------------------|:------------------|
| **Description**          | Raw disk access using illegitimate tools, possible defence evasion |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1006: Direct Volume Access](https://attack.mitre.org/techniques/T1006)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0013_9_windows_sysmon_RawAccessRead](../Data_Needed/DN_0013_9_windows_sysmon_RawAccessRead.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Legitimate Administrator using tool for raw access or ongoing forensic investigation</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment](https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment)</li></ul>  |
| **Author**               | Teymur Kheirkhabarov, oscd.community |


## Detection Rules

### Sigma rule

```
title: Raw Disk Access Using Illegitimate Tools
id: db809f10-56ce-4420-8c86-d6a7d793c79c
description: Raw disk access using illegitimate tools, possible defence evasion
author: Teymur Kheirkhabarov, oscd.community
date: 2019/10/22
references:
    - https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment
tags:
    - attack.defense_evasion
    - attack.t1006
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 9
    filter_1:
        Device|contains: floppy
    filter_2:
      - Image|endswith:             # easy to bypass. requires extra rule to support this one
            - '\wmiprvse.exe'
            - '\sdiagnhost.exe'
            - '\searchindexer.exe'
            - '\csrss.exe'
            - '\defrag.exe'
            - '\smss.exe'
            - '\vssvc.exe'
            - '\compattelrunner.exe'
            - '\wininit.exe'
            - '\autochk.exe'
            - '\taskhost.exe'
            - '\dfsrs.exe'
            - '\vds.exe'
            - '\lsass.exe'
    condition: selection and not filter_1 and not filter_2
fields:
    - ComputerName
    - Image
    - ProcessID
    - Device
falsepositives:
    - Legitimate Administrator using tool for raw access or ongoing forensic investigation
level: medium
status: experimental

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {(($_.ID -eq "9" -and  -not ($_.message -match "Device.*.*floppy.*")) -and  -not (($_.message -match "Image.*.*\\\\wmiprvse.exe" -or $_.message -match "Image.*.*\\\\sdiagnhost.exe" -or $_.message -match "Image.*.*\\\\searchindexer.exe" -or $_.message -match "Image.*.*\\\\csrss.exe" -or $_.message -match "Image.*.*\\\\defrag.exe" -or $_.message -match "Image.*.*\\\\smss.exe" -or $_.message -match "Image.*.*\\\\vssvc.exe" -or $_.message -match "Image.*.*\\\\compattelrunner.exe" -or $_.message -match "Image.*.*\\\\wininit.exe" -or $_.message -match "Image.*.*\\\\autochk.exe" -or $_.message -match "Image.*.*\\\\taskhost.exe" -or $_.message -match "Image.*.*\\\\dfsrs.exe" -or $_.message -match "Image.*.*\\\\vds.exe" -or $_.message -match "Image.*.*\\\\lsass.exe"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\\-Windows\\-Sysmon\\/Operational" AND (winlog.event_id:"9" AND (NOT (Device.keyword:*floppy*))) AND (NOT (winlog.event_data.Image.keyword:(*\\\\wmiprvse.exe OR *\\\\sdiagnhost.exe OR *\\\\searchindexer.exe OR *\\\\csrss.exe OR *\\\\defrag.exe OR *\\\\smss.exe OR *\\\\vssvc.exe OR *\\\\compattelrunner.exe OR *\\\\wininit.exe OR *\\\\autochk.exe OR *\\\\taskhost.exe OR *\\\\dfsrs.exe OR *\\\\vds.exe OR *\\\\lsass.exe))))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/db809f10-56ce-4420-8c86-d6a7d793c79c <<EOF\n{\n  "metadata": {\n    "title": "Raw Disk Access Using Illegitimate Tools",\n    "description": "Raw disk access using illegitimate tools, possible defence evasion",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1006"\n    ],\n    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND (winlog.event_id:\\"9\\" AND (NOT (Device.keyword:*floppy*))) AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\wmiprvse.exe OR *\\\\\\\\sdiagnhost.exe OR *\\\\\\\\searchindexer.exe OR *\\\\\\\\csrss.exe OR *\\\\\\\\defrag.exe OR *\\\\\\\\smss.exe OR *\\\\\\\\vssvc.exe OR *\\\\\\\\compattelrunner.exe OR *\\\\\\\\wininit.exe OR *\\\\\\\\autochk.exe OR *\\\\\\\\taskhost.exe OR *\\\\\\\\dfsrs.exe OR *\\\\\\\\vds.exe OR *\\\\\\\\lsass.exe))))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND (winlog.event_id:\\"9\\" AND (NOT (Device.keyword:*floppy*))) AND (NOT (winlog.event_data.Image.keyword:(*\\\\\\\\wmiprvse.exe OR *\\\\\\\\sdiagnhost.exe OR *\\\\\\\\searchindexer.exe OR *\\\\\\\\csrss.exe OR *\\\\\\\\defrag.exe OR *\\\\\\\\smss.exe OR *\\\\\\\\vssvc.exe OR *\\\\\\\\compattelrunner.exe OR *\\\\\\\\wininit.exe OR *\\\\\\\\autochk.exe OR *\\\\\\\\taskhost.exe OR *\\\\\\\\dfsrs.exe OR *\\\\\\\\vds.exe OR *\\\\\\\\lsass.exe))))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Raw Disk Access Using Illegitimate Tools\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\nComputerName = {{_source.ComputerName}}\\n       Image = {{_source.Image}}\\n   ProcessID = {{_source.ProcessID}}\\n      Device = {{_source.Device}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((EventID:"9" AND (NOT (Device.keyword:*floppy*))) AND (NOT (Image.keyword:(*\\\\wmiprvse.exe *\\\\sdiagnhost.exe *\\\\searchindexer.exe *\\\\csrss.exe *\\\\defrag.exe *\\\\smss.exe *\\\\vssvc.exe *\\\\compattelrunner.exe *\\\\wininit.exe *\\\\autochk.exe *\\\\taskhost.exe *\\\\dfsrs.exe *\\\\vds.exe *\\\\lsass.exe))))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode="9" NOT (Device="*floppy*")) NOT ((Image="*\\\\wmiprvse.exe" OR Image="*\\\\sdiagnhost.exe" OR Image="*\\\\searchindexer.exe" OR Image="*\\\\csrss.exe" OR Image="*\\\\defrag.exe" OR Image="*\\\\smss.exe" OR Image="*\\\\vssvc.exe" OR Image="*\\\\compattelrunner.exe" OR Image="*\\\\wininit.exe" OR Image="*\\\\autochk.exe" OR Image="*\\\\taskhost.exe" OR Image="*\\\\dfsrs.exe" OR Image="*\\\\vds.exe" OR Image="*\\\\lsass.exe"))) | table ComputerName,Image,ProcessID,Device
```


### logpoint
    
```
((event_id="9"  -(Device="*floppy*"))  -(Image IN ["*\\\\wmiprvse.exe", "*\\\\sdiagnhost.exe", "*\\\\searchindexer.exe", "*\\\\csrss.exe", "*\\\\defrag.exe", "*\\\\smss.exe", "*\\\\vssvc.exe", "*\\\\compattelrunner.exe", "*\\\\wininit.exe", "*\\\\autochk.exe", "*\\\\taskhost.exe", "*\\\\dfsrs.exe", "*\\\\vds.exe", "*\\\\lsass.exe"]))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*9)(?=.*(?!.*(?:.*(?=.*.*floppy.*))))))(?=.*(?!.*(?:.*(?:.*(?=.*(?:.*.*\\wmiprvse\\.exe|.*.*\\sdiagnhost\\.exe|.*.*\\searchindexer\\.exe|.*.*\\csrss\\.exe|.*.*\\defrag\\.exe|.*.*\\smss\\.exe|.*.*\\vssvc\\.exe|.*.*\\compattelrunner\\.exe|.*.*\\wininit\\.exe|.*.*\\autochk\\.exe|.*.*\\taskhost\\.exe|.*.*\\dfsrs\\.exe|.*.*\\vds\\.exe|.*.*\\lsass\\.exe)))))))'
```



