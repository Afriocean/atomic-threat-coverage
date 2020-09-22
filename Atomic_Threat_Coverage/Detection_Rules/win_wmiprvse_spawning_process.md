| Title                    | Wmiprvse Spawning Process       |
|:-------------------------|:------------------|
| **Description**          | Detects wmiprvse spawning processes |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1047: Windows Management Instrumentation](https://attack.mitre.org/techniques/T1047)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1047: Windows Management Instrumentation](../Triggers/T1047.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1047_windows_management_instrumentation/wmi_win32_process_create_remote.md](https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1047_windows_management_instrumentation/wmi_win32_process_create_remote.md)</li></ul>  |
| **Author**               | Roberto Rodriguez @Cyb3rWard0g |


## Detection Rules

### Sigma rule

```
title: Wmiprvse Spawning Process
id: d21374ff-f574-44a7-9998-4a8c8bf33d7d
description: Detects wmiprvse spawning processes
status: experimental
date: 2019/08/15
modified: 2019/11/10
author: Roberto Rodriguez @Cyb3rWard0g
references:
    - https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1047_windows_management_instrumentation/wmi_win32_process_create_remote.md
tags:
    - attack.execution
    - attack.t1047
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        ParentImage|endswith: '\WmiPrvSe.exe'
    filter:
        - LogonId: '0x3e7'  # LUID 999 for SYSTEM
        - User: 'NT AUTHORITY\SYSTEM'  # if we don't have LogonId data, fallback on username detection
    condition: selection and not filter
falsepositives:
    - Unknown
level: critical

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "ParentImage.*.*\\\\WmiPrvSe.exe" -and  -not ($_.message -match "LogonId.*0x3e7" -or $_.message -match "User.*NT AUTHORITY\\\\SYSTEM")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.ParentImage.keyword:*\\\\WmiPrvSe.exe AND (NOT (LogonId:"0x3e7" OR winlog.event_data.User:"NT\\ AUTHORITY\\\\SYSTEM")))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/d21374ff-f574-44a7-9998-4a8c8bf33d7d <<EOF\n{\n  "metadata": {\n    "title": "Wmiprvse Spawning Process",\n    "description": "Detects wmiprvse spawning processes",\n    "tags": [\n      "attack.execution",\n      "attack.t1047"\n    ],\n    "query": "(winlog.event_data.ParentImage.keyword:*\\\\\\\\WmiPrvSe.exe AND (NOT (LogonId:\\"0x3e7\\" OR winlog.event_data.User:\\"NT\\\\ AUTHORITY\\\\\\\\SYSTEM\\")))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.ParentImage.keyword:*\\\\\\\\WmiPrvSe.exe AND (NOT (LogonId:\\"0x3e7\\" OR winlog.event_data.User:\\"NT\\\\ AUTHORITY\\\\\\\\SYSTEM\\")))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Wmiprvse Spawning Process\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(ParentImage.keyword:*\\\\WmiPrvSe.exe AND (NOT (LogonId:"0x3e7" OR User:"NT AUTHORITY\\\\SYSTEM")))
```


### splunk
    
```
(ParentImage="*\\\\WmiPrvSe.exe" NOT (LogonId="0x3e7" OR User="NT AUTHORITY\\\\SYSTEM"))
```


### logpoint
    
```
(ParentImage="*\\\\WmiPrvSe.exe"  -(LogonId="0x3e7" OR User="NT AUTHORITY\\\\SYSTEM"))
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\WmiPrvSe\\.exe)(?=.*(?!.*(?:.*(?:.*(?=.*0x3e7)|.*(?=.*NT AUTHORITY\\SYSTEM))))))'
```



