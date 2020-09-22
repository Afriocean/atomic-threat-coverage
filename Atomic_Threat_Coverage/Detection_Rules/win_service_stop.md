| Title                    | Stop Windows Service       |
|:-------------------------|:------------------|
| **Description**          | Detects a windows service to be stopped |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0040: Impact](https://attack.mitre.org/tactics/TA0040)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1489: Service Stop](https://attack.mitre.org/techniques/T1489)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1489: Service Stop](../Triggers/T1489.md)</li></ul>  |
| **Severity Level**       | low |
| **False Positives**      | <ul><li>Administrator shutting down the service due to upgrade or removal purposes</li></ul>  |
| **Development Status**   | experimental |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Jakob Weinzettl, oscd.community |


## Detection Rules

### Sigma rule

```
title: Stop Windows Service
id: eb87818d-db5d-49cc-a987-d5da331fbd90
description: Detects a windows service to be stopped
status: experimental
author: Jakob Weinzettl, oscd.community
date: 2019/10/23
modified: 2019/11/08
tags:
    - attack.impact
    - attack.t1489
logsource:
    category: process_creation
    product: windows
detection:
    selection:
      - Image|endswith:
            - '\sc.exe'
            - '\net.exe'
            - '\net1.exe'
        CommandLine|contains: 'stop'
    condition: selection
fields:
    - ComputerName
    - User
    - CommandLine
falsepositives:
    - Administrator shutting down the service due to upgrade or removal purposes
level: low

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "Image.*.*\\\\sc.exe" -or $_.message -match "Image.*.*\\\\net.exe" -or $_.message -match "Image.*.*\\\\net1.exe") -and $_.message -match "CommandLine.*.*stop.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:(*\\\\sc.exe OR *\\\\net.exe OR *\\\\net1.exe) AND winlog.event_data.CommandLine.keyword:*stop*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/eb87818d-db5d-49cc-a987-d5da331fbd90 <<EOF\n{\n  "metadata": {\n    "title": "Stop Windows Service",\n    "description": "Detects a windows service to be stopped",\n    "tags": [\n      "attack.impact",\n      "attack.t1489"\n    ],\n    "query": "(winlog.event_data.Image.keyword:(*\\\\\\\\sc.exe OR *\\\\\\\\net.exe OR *\\\\\\\\net1.exe) AND winlog.event_data.CommandLine.keyword:*stop*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:(*\\\\\\\\sc.exe OR *\\\\\\\\net.exe OR *\\\\\\\\net1.exe) AND winlog.event_data.CommandLine.keyword:*stop*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Stop Windows Service\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\nComputerName = {{_source.ComputerName}}\\n        User = {{_source.User}}\\n CommandLine = {{_source.CommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:(*\\\\sc.exe *\\\\net.exe *\\\\net1.exe) AND CommandLine.keyword:*stop*)
```


### splunk
    
```
((Image="*\\\\sc.exe" OR Image="*\\\\net.exe" OR Image="*\\\\net1.exe") CommandLine="*stop*") | table ComputerName,User,CommandLine
```


### logpoint
    
```
(Image IN ["*\\\\sc.exe", "*\\\\net.exe", "*\\\\net1.exe"] CommandLine="*stop*")
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*.*\\sc\\.exe|.*.*\\net\\.exe|.*.*\\net1\\.exe))(?=.*.*stop.*))'
```



