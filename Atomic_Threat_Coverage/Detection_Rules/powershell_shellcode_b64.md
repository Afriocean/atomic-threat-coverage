| Title                    | PowerShell ShellCode       |
|:-------------------------|:------------------|
| **Description**          | Detects Base64 encoded Shellcode |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0004: Privilege Escalation](https://attack.mitre.org/tactics/TA0004)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1055: Process Injection](https://attack.mitre.org/techniques/T1055)</li><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059.001)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0036_4104_windows_powershell_script_block](../Data_Needed/DN_0036_4104_windows_powershell_script_block.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1055: Process Injection](../Triggers/T1055.md)</li><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://twitter.com/cyb3rops/status/1063072865992523776](https://twitter.com/cyb3rops/status/1063072865992523776)</li></ul>  |
| **Author**               | David Ledbetter (shellcode), Florian Roth (rule) |


## Detection Rules

### Sigma rule

```
title: PowerShell ShellCode
id: 16b37b70-6fcf-4814-a092-c36bd3aafcbd
status: experimental
description: Detects Base64 encoded Shellcode
references:
    - https://twitter.com/cyb3rops/status/1063072865992523776
tags:
    - attack.defense_evasion
    - attack.privilege_escalation
    - attack.t1055
    - attack.execution
    - attack.t1059.001
    - attack.t1086  #an old one
author: David Ledbetter (shellcode), Florian Roth (rule)
date: 2018/11/17
modified: 2020/08/24
logsource:
    product: windows
    service: powershell
    definition: 'Script block logging must be enabled'
detection:
    selection:
        EventID: 4104
    keyword1:
        - '*AAAAYInlM*'
    keyword2:
        - '*OiCAAAAYInlM*'
        - '*OiJAAAAYInlM*'
    condition: selection and keyword1 and keyword2
falsepositives:
    - Unknown
level: critical

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-PowerShell/Operational | where {(($_.ID -eq "4104" -and $_.message -match "*AAAAYInlM*") -and ($_.message -match "*OiCAAAAYInlM*" -or $_.message -match "*OiJAAAAYInlM*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_id:"4104" AND "*AAAAYInlM*") AND \\*.keyword:(*OiCAAAAYInlM* OR *OiJAAAAYInlM*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/16b37b70-6fcf-4814-a092-c36bd3aafcbd <<EOF\n{\n  "metadata": {\n    "title": "PowerShell ShellCode",\n    "description": "Detects Base64 encoded Shellcode",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.privilege_escalation",\n      "attack.t1055",\n      "attack.execution",\n      "attack.t1059.001",\n      "attack.t1086"\n    ],\n    "query": "((winlog.event_id:\\"4104\\" AND \\"*AAAAYInlM*\\") AND \\\\*.keyword:(*OiCAAAAYInlM* OR *OiJAAAAYInlM*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_id:\\"4104\\" AND \\"*AAAAYInlM*\\") AND \\\\*.keyword:(*OiCAAAAYInlM* OR *OiJAAAAYInlM*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'PowerShell ShellCode\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((EventID:"4104" AND "*AAAAYInlM*") AND \\*.keyword:(*OiCAAAAYInlM* OR *OiJAAAAYInlM*))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-PowerShell/Operational" (EventCode="4104" "*AAAAYInlM*") ("*OiCAAAAYInlM*" OR "*OiJAAAAYInlM*"))
```


### logpoint
    
```
((event_id="4104" "*AAAAYInlM*") ("*OiCAAAAYInlM*" OR "*OiJAAAAYInlM*"))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*4104)(?=.*.*AAAAYInlM.*)))(?=.*(?:.*(?:.*.*OiCAAAAYInlM.*|.*.*OiJAAAAYInlM.*))))'
```



