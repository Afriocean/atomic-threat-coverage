| Title                    | Psr.exe Capture Screenshots       |
|:-------------------------|:------------------|
| **Description**          | The psr.exe captures desktop screenshots and saves them on the local machine |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0009: Collection](https://attack.mitre.org/tactics/TA0009)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1113: Screen Capture](https://attack.mitre.org/techniques/T1113)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1113: Screen Capture](../Triggers/T1113.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/LOLBAS-Project/LOLBAS/blob/master/yml/LOLUtilz/OSBinaries/Psr.yml](https://github.com/LOLBAS-Project/LOLBAS/blob/master/yml/LOLUtilz/OSBinaries/Psr.yml)</li><li>[https://www.sans.org/summit-archives/file/summit-archive-1493861893.pdf](https://www.sans.org/summit-archives/file/summit-archive-1493861893.pdf)</li></ul>  |
| **Author**               | Beyu Denis, oscd.community |


## Detection Rules

### Sigma rule

```
title: Psr.exe Capture Screenshots
id: 2158f96f-43c2-43cb-952a-ab4580f32382
status: experimental
description: The psr.exe captures desktop screenshots and saves them on the local machine
references:
    - https://github.com/LOLBAS-Project/LOLBAS/blob/master/yml/LOLUtilz/OSBinaries/Psr.yml
    - https://www.sans.org/summit-archives/file/summit-archive-1493861893.pdf
author: Beyu Denis, oscd.community
date: 2019/10/12
modified: 2020/08/28
tags:
    - attack.collection
    - attack.t1113
level: medium
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\Psr.exe'
        CommandLine|contains: '/start'
    condition: selection
falsepositives:
    - Unknown

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\\\Psr.exe" -and $_.message -match "CommandLine.*.*/start.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\\\Psr.exe AND winlog.event_data.CommandLine.keyword:*\\/start*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/2158f96f-43c2-43cb-952a-ab4580f32382 <<EOF\n{\n  "metadata": {\n    "title": "Psr.exe Capture Screenshots",\n    "description": "The psr.exe captures desktop screenshots and saves them on the local machine",\n    "tags": [\n      "attack.collection",\n      "attack.t1113"\n    ],\n    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\Psr.exe AND winlog.event_data.CommandLine.keyword:*\\\\/start*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\Psr.exe AND winlog.event_data.CommandLine.keyword:*\\\\/start*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Psr.exe Capture Screenshots\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:*\\\\Psr.exe AND CommandLine.keyword:*\\/start*)
```


### splunk
    
```
(Image="*\\\\Psr.exe" CommandLine="*/start*")
```


### logpoint
    
```
(Image="*\\\\Psr.exe" CommandLine="*/start*")
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\Psr\\.exe)(?=.*.*/start.*))'
```



