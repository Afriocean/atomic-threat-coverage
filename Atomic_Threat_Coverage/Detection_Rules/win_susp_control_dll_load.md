| Title                    | Suspicious Control Panel DLL Load       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious Rundll32 execution from control.exe as used by Equation Group and Exploit Kits |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1085: Rundll32](https://attack.mitre.org/techniques/T1085)</li><li>[T1218.011: Rundll32](https://attack.mitre.org/techniques/T1218.011)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1218.011: Rundll32](../Triggers/T1218.011.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://twitter.com/rikvduijn/status/853251879320662017](https://twitter.com/rikvduijn/status/853251879320662017)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Suspicious Control Panel DLL Load
id: d7eb979b-c2b5-4a6f-a3a7-c87ce6763819
status: experimental
description: Detects suspicious Rundll32 execution from control.exe as used by Equation Group and Exploit Kits
author: Florian Roth
date: 2017/04/15
modified: 2020/09/05
references:
    - https://twitter.com/rikvduijn/status/853251879320662017
tags:
    - attack.defense_evasion
    - attack.t1085      # an old one
    - attack.t1218.011
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        ParentImage: '*\System32\control.exe'
        CommandLine: '*\rundll32.exe *'
    filter:
        CommandLine: '*Shell32.dll*'
    condition: selection and not filter
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "ParentImage.*.*\\\\System32\\\\control.exe" -and $_.message -match "CommandLine.*.*\\\\rundll32.exe .*") -and  -not ($_.message -match "CommandLine.*.*Shell32.dll.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.ParentImage.keyword:*\\\\System32\\\\control.exe AND winlog.event_data.CommandLine.keyword:*\\\\rundll32.exe\\ *) AND (NOT (winlog.event_data.CommandLine.keyword:*Shell32.dll*)))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/d7eb979b-c2b5-4a6f-a3a7-c87ce6763819 <<EOF\n{\n  "metadata": {\n    "title": "Suspicious Control Panel DLL Load",\n    "description": "Detects suspicious Rundll32 execution from control.exe as used by Equation Group and Exploit Kits",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1085",\n      "attack.t1218.011"\n    ],\n    "query": "((winlog.event_data.ParentImage.keyword:*\\\\\\\\System32\\\\\\\\control.exe AND winlog.event_data.CommandLine.keyword:*\\\\\\\\rundll32.exe\\\\ *) AND (NOT (winlog.event_data.CommandLine.keyword:*Shell32.dll*)))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.ParentImage.keyword:*\\\\\\\\System32\\\\\\\\control.exe AND winlog.event_data.CommandLine.keyword:*\\\\\\\\rundll32.exe\\\\ *) AND (NOT (winlog.event_data.CommandLine.keyword:*Shell32.dll*)))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Suspicious Control Panel DLL Load\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((ParentImage.keyword:*\\\\System32\\\\control.exe AND CommandLine.keyword:*\\\\rundll32.exe *) AND (NOT (CommandLine.keyword:*Shell32.dll*)))
```


### splunk
    
```
((ParentImage="*\\\\System32\\\\control.exe" CommandLine="*\\\\rundll32.exe *") NOT (CommandLine="*Shell32.dll*")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
((ParentImage="*\\\\System32\\\\control.exe" CommandLine="*\\\\rundll32.exe *")  -(CommandLine="*Shell32.dll*"))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*.*\\System32\\control\\.exe)(?=.*.*\\rundll32\\.exe .*)))(?=.*(?!.*(?:.*(?=.*.*Shell32\\.dll.*)))))'
```



