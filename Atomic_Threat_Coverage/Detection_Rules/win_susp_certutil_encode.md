| Title                    | Certutil Encode       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious a certutil command that used to encode files, which is sometimes used for data exfiltration |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1027: Obfuscated Files or Information](https://attack.mitre.org/techniques/T1027)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1027: Obfuscated Files or Information](../Triggers/T1027.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil)</li><li>[https://unit42.paloaltonetworks.com/new-babyshark-malware-targets-u-s-national-security-think-tanks/](https://unit42.paloaltonetworks.com/new-babyshark-malware-targets-u-s-national-security-think-tanks/)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Certutil Encode
id: e62a9f0c-ca1e-46b2-85d5-a6da77f86d1a
status: experimental
description: Detects suspicious a certutil command that used to encode files, which is sometimes used for data exfiltration
references:
    - https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil
    - https://unit42.paloaltonetworks.com/new-babyshark-malware-targets-u-s-national-security-think-tanks/
author: Florian Roth
date: 2019/02/24
modified: 2020/09/05
tags:
    - attack.defense_evasion
    - attack.t1027
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine:
            - certutil -f -encode *
            - certutil.exe -f -encode *
            - certutil -encode -f *
            - certutil.exe -encode -f *
    condition: selection
falsepositives:
    - unknown
level: medium

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "CommandLine.*certutil -f -encode .*" -or $_.message -match "CommandLine.*certutil.exe -f -encode .*" -or $_.message -match "CommandLine.*certutil -encode -f .*" -or $_.message -match "CommandLine.*certutil.exe -encode -f .*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:(certutil\\ \\-f\\ \\-encode\\ * OR certutil.exe\\ \\-f\\ \\-encode\\ * OR certutil\\ \\-encode\\ \\-f\\ * OR certutil.exe\\ \\-encode\\ \\-f\\ *)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/e62a9f0c-ca1e-46b2-85d5-a6da77f86d1a <<EOF\n{\n  "metadata": {\n    "title": "Certutil Encode",\n    "description": "Detects suspicious a certutil command that used to encode files, which is sometimes used for data exfiltration",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1027"\n    ],\n    "query": "winlog.event_data.CommandLine.keyword:(certutil\\\\ \\\\-f\\\\ \\\\-encode\\\\ * OR certutil.exe\\\\ \\\\-f\\\\ \\\\-encode\\\\ * OR certutil\\\\ \\\\-encode\\\\ \\\\-f\\\\ * OR certutil.exe\\\\ \\\\-encode\\\\ \\\\-f\\\\ *)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "winlog.event_data.CommandLine.keyword:(certutil\\\\ \\\\-f\\\\ \\\\-encode\\\\ * OR certutil.exe\\\\ \\\\-f\\\\ \\\\-encode\\\\ * OR certutil\\\\ \\\\-encode\\\\ \\\\-f\\\\ * OR certutil.exe\\\\ \\\\-encode\\\\ \\\\-f\\\\ *)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Certutil Encode\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
CommandLine.keyword:(certutil \\-f \\-encode * certutil.exe \\-f \\-encode * certutil \\-encode \\-f * certutil.exe \\-encode \\-f *)
```


### splunk
    
```
(CommandLine="certutil -f -encode *" OR CommandLine="certutil.exe -f -encode *" OR CommandLine="certutil -encode -f *" OR CommandLine="certutil.exe -encode -f *")
```


### logpoint
    
```
CommandLine IN ["certutil -f -encode *", "certutil.exe -f -encode *", "certutil -encode -f *", "certutil.exe -encode -f *"]
```


### grep
    
```
grep -P '^(?:.*certutil -f -encode .*|.*certutil\\.exe -f -encode .*|.*certutil -encode -f .*|.*certutil\\.exe -encode -f .*)'
```



