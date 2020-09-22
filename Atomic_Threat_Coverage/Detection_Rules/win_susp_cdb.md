| Title                    | Possible App Whitelisting Bypass via WinDbg/CDB as a Shellcode Runner       |
|:-------------------------|:------------------|
| **Description**          | Launch 64-bit shellcode from a debugger script file using cdb.exe. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1106: Native API](https://attack.mitre.org/techniques/T1106)</li><li>[T1218: Signed Binary Proxy Execution](https://attack.mitre.org/techniques/T1218)</li><li>[T1127: Trusted Developer Utilities Proxy Execution](https://attack.mitre.org/techniques/T1127)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1106: Native API](../Triggers/T1106.md)</li><li>[T1218: Signed Binary Proxy Execution](../Triggers/T1218.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Legitimate use of debugging tools</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/LOLBAS-Project/LOLBAS/blob/master/yml/OtherMSBinaries/Cdb.yml](https://github.com/LOLBAS-Project/LOLBAS/blob/master/yml/OtherMSBinaries/Cdb.yml)</li><li>[http://www.exploit-monday.com/2016/08/windbg-cdb-shellcode-runner.html](http://www.exploit-monday.com/2016/08/windbg-cdb-shellcode-runner.html)</li></ul>  |
| **Author**               | Beyu Denis, oscd.community |


## Detection Rules

### Sigma rule

```
title: Possible App Whitelisting Bypass via WinDbg/CDB as a Shellcode Runner
id: b5c7395f-e501-4a08-94d4-57fe7a9da9d2
status: experimental
description: Launch 64-bit shellcode from a debugger script file using cdb.exe.
references:
    - https://github.com/LOLBAS-Project/LOLBAS/blob/master/yml/OtherMSBinaries/Cdb.yml
    - http://www.exploit-monday.com/2016/08/windbg-cdb-shellcode-runner.html
author: Beyu Denis, oscd.community
date: 2019/10/26
modified: 2020/09/05
tags:
    - attack.execution
    - attack.t1106
    - attack.defense_evasion
    - attack.t1218
    - attack.t1127
level: medium
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\cdb.exe'
        CommandLine|contains: '-cf'
    condition: selection
falsepositives:
    - Legitimate use of debugging tools

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\\\cdb.exe" -and $_.message -match "CommandLine.*.*-cf.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\\\cdb.exe AND winlog.event_data.CommandLine.keyword:*\\-cf*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/b5c7395f-e501-4a08-94d4-57fe7a9da9d2 <<EOF\n{\n  "metadata": {\n    "title": "Possible App Whitelisting Bypass via WinDbg/CDB as a Shellcode Runner",\n    "description": "Launch 64-bit shellcode from a debugger script file using cdb.exe.",\n    "tags": [\n      "attack.execution",\n      "attack.t1106",\n      "attack.defense_evasion",\n      "attack.t1218",\n      "attack.t1127"\n    ],\n    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\cdb.exe AND winlog.event_data.CommandLine.keyword:*\\\\-cf*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\cdb.exe AND winlog.event_data.CommandLine.keyword:*\\\\-cf*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Possible App Whitelisting Bypass via WinDbg/CDB as a Shellcode Runner\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:*\\\\cdb.exe AND CommandLine.keyword:*\\-cf*)
```


### splunk
    
```
(Image="*\\\\cdb.exe" CommandLine="*-cf*")
```


### logpoint
    
```
(Image="*\\\\cdb.exe" CommandLine="*-cf*")
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\cdb\\.exe)(?=.*.*-cf.*))'
```



