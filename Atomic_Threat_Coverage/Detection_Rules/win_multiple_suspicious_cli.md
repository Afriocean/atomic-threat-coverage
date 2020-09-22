| Title                    | Quick Execution of a Series of Suspicious Commands       |
|:-------------------------|:------------------|
| **Description**          | Detects multiple suspicious process in a limited timeframe |
| **ATT&amp;CK Tactic**    |   This Detection Rule wasn't mapped to ATT&amp;CK Tactic yet  |
| **ATT&amp;CK Technique** |  This Detection Rule wasn't mapped to ATT&amp;CK Technique yet  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | low |
| **False Positives**      | <ul><li>False positives depend on scripts and administrative tools used in the monitored environment</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://car.mitre.org/wiki/CAR-2013-04-002](https://car.mitre.org/wiki/CAR-2013-04-002)</li></ul>  |
| **Author**               | juju4 |
| Other Tags           | <ul><li>car.2013-04-002</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Quick Execution of a Series of Suspicious Commands
id: 61ab5496-748e-4818-a92f-de78e20fe7f1
description: Detects multiple suspicious process in a limited timeframe
status: experimental
references:
    - https://car.mitre.org/wiki/CAR-2013-04-002
author: juju4
date: 2019/01/16
tags:
    - car.2013-04-002
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine:
            - arp.exe
            - at.exe
            - attrib.exe
            - cscript.exe
            - dsquery.exe
            - hostname.exe
            - ipconfig.exe
            - mimikatz.exe
            - nbtstat.exe
            - net.exe
            - netsh.exe
            - nslookup.exe
            - ping.exe
            - quser.exe
            - qwinsta.exe
            - reg.exe
            - runas.exe
            - sc.exe
            - schtasks.exe
            - ssh.exe
            - systeminfo.exe
            - taskkill.exe
            - telnet.exe
            - tracert.exe
            - wscript.exe
            - xcopy.exe
            - pscp.exe
            - copy.exe
            - robocopy.exe
            - certutil.exe
            - vssadmin.exe
            - powershell.exe
            - wevtutil.exe
            - psexec.exe
            - bcedit.exe
            - wbadmin.exe
            - icacls.exe
            - diskpart.exe
    timeframe: 5m
    condition: selection | count() by MachineName > 5
falsepositives:
    - False positives depend on scripts and administrative tools used in the monitored environment
level: low

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "arp.exe" -or $_.message -match "at.exe" -or $_.message -match "attrib.exe" -or $_.message -match "cscript.exe" -or $_.message -match "dsquery.exe" -or $_.message -match "hostname.exe" -or $_.message -match "ipconfig.exe" -or $_.message -match "mimikatz.exe" -or $_.message -match "nbtstat.exe" -or $_.message -match "net.exe" -or $_.message -match "netsh.exe" -or $_.message -match "nslookup.exe" -or $_.message -match "ping.exe" -or $_.message -match "quser.exe" -or $_.message -match "qwinsta.exe" -or $_.message -match "reg.exe" -or $_.message -match "runas.exe" -or $_.message -match "sc.exe" -or $_.message -match "schtasks.exe" -or $_.message -match "ssh.exe" -or $_.message -match "systeminfo.exe" -or $_.message -match "taskkill.exe" -or $_.message -match "telnet.exe" -or $_.message -match "tracert.exe" -or $_.message -match "wscript.exe" -or $_.message -match "xcopy.exe" -or $_.message -match "pscp.exe" -or $_.message -match "copy.exe" -or $_.message -match "robocopy.exe" -or $_.message -match "certutil.exe" -or $_.message -match "vssadmin.exe" -or $_.message -match "powershell.exe" -or $_.message -match "wevtutil.exe" -or $_.message -match "psexec.exe" -or $_.message -match "bcedit.exe" -or $_.message -match "wbadmin.exe" -or $_.message -match "icacls.exe" -or $_.message -match "diskpart.exe") }  | group-object MachineName | where { $_.count -gt 5 } | select name,count | sort -desc
```


### es-qs
    
```

```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/61ab5496-748e-4818-a92f-de78e20fe7f1 <<EOF\n{\n  "metadata": {\n    "title": "Quick Execution of a Series of Suspicious Commands",\n    "description": "Detects multiple suspicious process in a limited timeframe",\n    "tags": [\n      "car.2013-04-002"\n    ],\n    "query": "winlog.event_data.CommandLine:(\\"arp.exe\\" OR \\"at.exe\\" OR \\"attrib.exe\\" OR \\"cscript.exe\\" OR \\"dsquery.exe\\" OR \\"hostname.exe\\" OR \\"ipconfig.exe\\" OR \\"mimikatz.exe\\" OR \\"nbtstat.exe\\" OR \\"net.exe\\" OR \\"netsh.exe\\" OR \\"nslookup.exe\\" OR \\"ping.exe\\" OR \\"quser.exe\\" OR \\"qwinsta.exe\\" OR \\"reg.exe\\" OR \\"runas.exe\\" OR \\"sc.exe\\" OR \\"schtasks.exe\\" OR \\"ssh.exe\\" OR \\"systeminfo.exe\\" OR \\"taskkill.exe\\" OR \\"telnet.exe\\" OR \\"tracert.exe\\" OR \\"wscript.exe\\" OR \\"xcopy.exe\\" OR \\"pscp.exe\\" OR \\"copy.exe\\" OR \\"robocopy.exe\\" OR \\"certutil.exe\\" OR \\"vssadmin.exe\\" OR \\"powershell.exe\\" OR \\"wevtutil.exe\\" OR \\"psexec.exe\\" OR \\"bcedit.exe\\" OR \\"wbadmin.exe\\" OR \\"icacls.exe\\" OR \\"diskpart.exe\\")"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "5m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "winlog.event_data.CommandLine:(\\"arp.exe\\" OR \\"at.exe\\" OR \\"attrib.exe\\" OR \\"cscript.exe\\" OR \\"dsquery.exe\\" OR \\"hostname.exe\\" OR \\"ipconfig.exe\\" OR \\"mimikatz.exe\\" OR \\"nbtstat.exe\\" OR \\"net.exe\\" OR \\"netsh.exe\\" OR \\"nslookup.exe\\" OR \\"ping.exe\\" OR \\"quser.exe\\" OR \\"qwinsta.exe\\" OR \\"reg.exe\\" OR \\"runas.exe\\" OR \\"sc.exe\\" OR \\"schtasks.exe\\" OR \\"ssh.exe\\" OR \\"systeminfo.exe\\" OR \\"taskkill.exe\\" OR \\"telnet.exe\\" OR \\"tracert.exe\\" OR \\"wscript.exe\\" OR \\"xcopy.exe\\" OR \\"pscp.exe\\" OR \\"copy.exe\\" OR \\"robocopy.exe\\" OR \\"certutil.exe\\" OR \\"vssadmin.exe\\" OR \\"powershell.exe\\" OR \\"wevtutil.exe\\" OR \\"psexec.exe\\" OR \\"bcedit.exe\\" OR \\"wbadmin.exe\\" OR \\"icacls.exe\\" OR \\"diskpart.exe\\")",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          },\n          "aggs": {\n            "by": {\n              "terms": {\n                "field": "MachineName",\n                "size": 10,\n                "order": {\n                  "_count": "desc"\n                },\n                "min_doc_count": 6\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.aggregations.by.buckets.0.doc_count": {\n        "gt": 5\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Quick Execution of a Series of Suspicious Commands\'",\n        "body": "Hits:\\n{{#aggregations.by.buckets}}\\n {{key}} {{doc_count}}\\n{{/aggregations.by.buckets}}\\n",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```

```


### splunk
    
```
(CommandLine="arp.exe" OR CommandLine="at.exe" OR CommandLine="attrib.exe" OR CommandLine="cscript.exe" OR CommandLine="dsquery.exe" OR CommandLine="hostname.exe" OR CommandLine="ipconfig.exe" OR CommandLine="mimikatz.exe" OR CommandLine="nbtstat.exe" OR CommandLine="net.exe" OR CommandLine="netsh.exe" OR CommandLine="nslookup.exe" OR CommandLine="ping.exe" OR CommandLine="quser.exe" OR CommandLine="qwinsta.exe" OR CommandLine="reg.exe" OR CommandLine="runas.exe" OR CommandLine="sc.exe" OR CommandLine="schtasks.exe" OR CommandLine="ssh.exe" OR CommandLine="systeminfo.exe" OR CommandLine="taskkill.exe" OR CommandLine="telnet.exe" OR CommandLine="tracert.exe" OR CommandLine="wscript.exe" OR CommandLine="xcopy.exe" OR CommandLine="pscp.exe" OR CommandLine="copy.exe" OR CommandLine="robocopy.exe" OR CommandLine="certutil.exe" OR CommandLine="vssadmin.exe" OR CommandLine="powershell.exe" OR CommandLine="wevtutil.exe" OR CommandLine="psexec.exe" OR CommandLine="bcedit.exe" OR CommandLine="wbadmin.exe" OR CommandLine="icacls.exe" OR CommandLine="diskpart.exe") | eventstats count as val by MachineName| search val > 5
```


### logpoint
    
```
CommandLine IN ["arp.exe", "at.exe", "attrib.exe", "cscript.exe", "dsquery.exe", "hostname.exe", "ipconfig.exe", "mimikatz.exe", "nbtstat.exe", "net.exe", "netsh.exe", "nslookup.exe", "ping.exe", "quser.exe", "qwinsta.exe", "reg.exe", "runas.exe", "sc.exe", "schtasks.exe", "ssh.exe", "systeminfo.exe", "taskkill.exe", "telnet.exe", "tracert.exe", "wscript.exe", "xcopy.exe", "pscp.exe", "copy.exe", "robocopy.exe", "certutil.exe", "vssadmin.exe", "powershell.exe", "wevtutil.exe", "psexec.exe", "bcedit.exe", "wbadmin.exe", "icacls.exe", "diskpart.exe"] | chart count() as val by MachineName | search val > 5
```


### grep
    
```
grep -P '^(?:.*arp\\.exe|.*at\\.exe|.*attrib\\.exe|.*cscript\\.exe|.*dsquery\\.exe|.*hostname\\.exe|.*ipconfig\\.exe|.*mimikatz\\.exe|.*nbtstat\\.exe|.*net\\.exe|.*netsh\\.exe|.*nslookup\\.exe|.*ping\\.exe|.*quser\\.exe|.*qwinsta\\.exe|.*reg\\.exe|.*runas\\.exe|.*sc\\.exe|.*schtasks\\.exe|.*ssh\\.exe|.*systeminfo\\.exe|.*taskkill\\.exe|.*telnet\\.exe|.*tracert\\.exe|.*wscript\\.exe|.*xcopy\\.exe|.*pscp\\.exe|.*copy\\.exe|.*robocopy\\.exe|.*certutil\\.exe|.*vssadmin\\.exe|.*powershell\\.exe|.*wevtutil\\.exe|.*psexec\\.exe|.*bcedit\\.exe|.*wbadmin\\.exe|.*icacls\\.exe|.*diskpart\\.exe)'
```



