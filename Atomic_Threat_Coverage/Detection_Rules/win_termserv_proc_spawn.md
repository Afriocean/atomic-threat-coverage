| Title                    | Terminal Service Process Spawn       |
|:-------------------------|:------------------|
| **Description**          | Detects a process spawned by the terminal service server process (this could be an indicator for an exploitation of CVE-2019-0708) |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0001: Initial Access](https://attack.mitre.org/tactics/TA0001)</li><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1190: Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190)</li><li>[T1210: Exploitation of Remote Services](https://attack.mitre.org/techniques/T1210)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://securingtomorrow.mcafee.com/other-blogs/mcafee-labs/rdp-stands-for-really-do-patch-understanding-the-wormable-rdp-vulnerability-cve-2019-0708/](https://securingtomorrow.mcafee.com/other-blogs/mcafee-labs/rdp-stands-for-really-do-patch-understanding-the-wormable-rdp-vulnerability-cve-2019-0708/)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>car.2013-07-002</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Terminal Service Process Spawn
id: 1012f107-b8f1-4271-af30-5aed2de89b39
status: experimental
description: Detects a process spawned by the terminal service server process (this could be an indicator for an exploitation of CVE-2019-0708)
references:
    - https://securingtomorrow.mcafee.com/other-blogs/mcafee-labs/rdp-stands-for-really-do-patch-understanding-the-wormable-rdp-vulnerability-cve-2019-0708/
author: Florian Roth
date: 2019/05/22
modified: 2020/08/29
tags:
    - attack.initial_access 
    - attack.t1190
    - attack.lateral_movement
    - attack.t1210
    - car.2013-07-002
logsource:
    product: windows
    category: process_creation
detection:
    selection:
        ParentCommandLine: '*\svchost.exe*termsvcs'
    filter:
        Image: '*\rdpclip.exe'
    condition: selection and not filter
falsepositives:
    - Unknown
level: high
```





### powershell
    
```
Get-WinEvent | where {($_.message -match "ParentCommandLine.*.*\\\\svchost.exe.*termsvcs" -and  -not ($_.message -match "Image.*.*\\\\rdpclip.exe")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.ParentCommandLine.keyword:*\\\\svchost.exe*termsvcs AND (NOT (winlog.event_data.Image.keyword:*\\\\rdpclip.exe)))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/1012f107-b8f1-4271-af30-5aed2de89b39 <<EOF\n{\n  "metadata": {\n    "title": "Terminal Service Process Spawn",\n    "description": "Detects a process spawned by the terminal service server process (this could be an indicator for an exploitation of CVE-2019-0708)",\n    "tags": [\n      "attack.initial_access",\n      "attack.t1190",\n      "attack.lateral_movement",\n      "attack.t1210",\n      "car.2013-07-002"\n    ],\n    "query": "(winlog.event_data.ParentCommandLine.keyword:*\\\\\\\\svchost.exe*termsvcs AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\rdpclip.exe)))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.ParentCommandLine.keyword:*\\\\\\\\svchost.exe*termsvcs AND (NOT (winlog.event_data.Image.keyword:*\\\\\\\\rdpclip.exe)))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Terminal Service Process Spawn\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(ParentCommandLine.keyword:*\\\\svchost.exe*termsvcs AND (NOT (Image.keyword:*\\\\rdpclip.exe)))
```


### splunk
    
```
(ParentCommandLine="*\\\\svchost.exe*termsvcs" NOT (Image="*\\\\rdpclip.exe"))
```


### logpoint
    
```
(ParentCommandLine="*\\\\svchost.exe*termsvcs"  -(Image="*\\\\rdpclip.exe"))
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\svchost\\.exe.*termsvcs)(?=.*(?!.*(?:.*(?=.*.*\\rdpclip\\.exe)))))'
```



