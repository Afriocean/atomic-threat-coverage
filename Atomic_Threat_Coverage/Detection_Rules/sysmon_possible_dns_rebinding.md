| Title                    | Possible DNS Rebinding       |
|:-------------------------|:------------------|
| **Description**          | Detects several different DNS-answers by one domain with IPs from internal and external networks. Normally, DNS-answer contain TTL >100. (DNS-record will saved in host cache for a while TTL). |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0001: Initial Access](https://attack.mitre.org/tactics/TA0001)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1189: Drive-by Compromise](https://attack.mitre.org/techniques/T1189)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0085_22_windows_sysmon_DnsQuery](../Data_Needed/DN_0085_22_windows_sysmon_DnsQuery.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | medium |
| **False Positives**      |  There are no documented False Positives for this Detection Rule yet  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://medium.com/@brannondorsey/attacking-private-networks-from-the-internet-with-dns-rebinding-ea7098a2d325](https://medium.com/@brannondorsey/attacking-private-networks-from-the-internet-with-dns-rebinding-ea7098a2d325)</li></ul>  |
| **Author**               | Ilyas Ochkov, oscd.community |


## Detection Rules

### Sigma rule

```
title: Possible DNS Rebinding
id: eb07e747-2552-44cd-af36-b659ae0958e4
status: experimental
description: Detects several different DNS-answers by one domain with IPs from internal and external networks. Normally, DNS-answer contain TTL >100. (DNS-record will saved in host cache for a while TTL).
date: 2019/10/25
modified: 2020/08/28
author: Ilyas Ochkov, oscd.community
references:
    - https://medium.com/@brannondorsey/attacking-private-networks-from-the-internet-with-dns-rebinding-ea7098a2d325
tags:
    - attack.initial_access
    - attack.t1189
logsource:
    product: windows
    service: sysmon
detection:
    dns_answer:
        EventID: 22
        QueryName: '*'
        QueryStatus: '0'
    filter_int_ip:
        QueryResults|startswith:
            - '(::ffff:)?10.'
            - '(::ffff:)?192.168.'
            - '(::ffff:)?172.16.'
            - '(::ffff:)?172.17.'
            - '(::ffff:)?172.18.'
            - '(::ffff:)?172.19.'
            - '(::ffff:)?172.20.'
            - '(::ffff:)?172.21.'
            - '(::ffff:)?172.22.'
            - '(::ffff:)?172.23.'
            - '(::ffff:)?172.24.'
            - '(::ffff:)?172.25.'
            - '(::ffff:)?172.26.'
            - '(::ffff:)?172.27.'
            - '(::ffff:)?172.28.'
            - '(::ffff:)?172.29.'
            - '(::ffff:)?172.30.'
            - '(::ffff:)?172.31.'
            - '(::ffff:)?127.'
    timeframe: 30s
    condition: (dns_answer and filter_int_ip) and (dns_answer and not filter_int_ip) | count(QueryName) by ComputerName > 3
level: medium

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "22" -and $_.message -match "QueryName.*.*" -and $_.message -match "QueryStatus.*0" -and ($_.message -match "QueryResults.*(::ffff:)?10..*" -or $_.message -match "QueryResults.*(::ffff:)?192.168..*" -or $_.message -match "QueryResults.*(::ffff:)?172.16..*" -or $_.message -match "QueryResults.*(::ffff:)?172.17..*" -or $_.message -match "QueryResults.*(::ffff:)?172.18..*" -or $_.message -match "QueryResults.*(::ffff:)?172.19..*" -or $_.message -match "QueryResults.*(::ffff:)?172.20..*" -or $_.message -match "QueryResults.*(::ffff:)?172.21..*" -or $_.message -match "QueryResults.*(::ffff:)?172.22..*" -or $_.message -match "QueryResults.*(::ffff:)?172.23..*" -or $_.message -match "QueryResults.*(::ffff:)?172.24..*" -or $_.message -match "QueryResults.*(::ffff:)?172.25..*" -or $_.message -match "QueryResults.*(::ffff:)?172.26..*" -or $_.message -match "QueryResults.*(::ffff:)?172.27..*" -or $_.message -match "QueryResults.*(::ffff:)?172.28..*" -or $_.message -match "QueryResults.*(::ffff:)?172.29..*" -or $_.message -match "QueryResults.*(::ffff:)?172.30..*" -or $_.message -match "QueryResults.*(::ffff:)?172.31..*" -or $_.message -match "QueryResults.*(::ffff:)?127..*") -and ($_.ID -eq "22" -and $_.message -match "QueryName.*.*" -and $_.message -match "QueryStatus.*0") -and  -not (($_.message -match "QueryResults.*(::ffff:)?10..*" -or $_.message -match "QueryResults.*(::ffff:)?192.168..*" -or $_.message -match "QueryResults.*(::ffff:)?172.16..*" -or $_.message -match "QueryResults.*(::ffff:)?172.17..*" -or $_.message -match "QueryResults.*(::ffff:)?172.18..*" -or $_.message -match "QueryResults.*(::ffff:)?172.19..*" -or $_.message -match "QueryResults.*(::ffff:)?172.20..*" -or $_.message -match "QueryResults.*(::ffff:)?172.21..*" -or $_.message -match "QueryResults.*(::ffff:)?172.22..*" -or $_.message -match "QueryResults.*(::ffff:)?172.23..*" -or $_.message -match "QueryResults.*(::ffff:)?172.24..*" -or $_.message -match "QueryResults.*(::ffff:)?172.25..*" -or $_.message -match "QueryResults.*(::ffff:)?172.26..*" -or $_.message -match "QueryResults.*(::ffff:)?172.27..*" -or $_.message -match "QueryResults.*(::ffff:)?172.28..*" -or $_.message -match "QueryResults.*(::ffff:)?172.29..*" -or $_.message -match "QueryResults.*(::ffff:)?172.30..*" -or $_.message -match "QueryResults.*(::ffff:)?172.31..*" -or $_.message -match "QueryResults.*(::ffff:)?127..*"))) }  | select ComputerName, QueryName | group ComputerName | foreach { [PSCustomObject]@{\'ComputerName\'=$_.name;\'Count\'=($_.group.QueryName | sort -u).count} }  | sort count -desc | where { $_.count -gt 3 }
```


### es-qs
    
```

```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/eb07e747-2552-44cd-af36-b659ae0958e4 <<EOF\n{\n  "metadata": {\n    "title": "Possible DNS Rebinding",\n    "description": "Detects several different DNS-answers by one domain with IPs from internal and external networks. Normally, DNS-answer contain TTL >100. (DNS-record will saved in host cache for a while TTL).",\n    "tags": [\n      "attack.initial_access",\n      "attack.t1189"\n    ],\n    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"22\\" AND QueryName.keyword:* AND QueryStatus:\\"0\\" AND QueryResults.keyword:(\\\\(\\\\:\\\\:ffff\\\\:\\\\)?10.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?192.168.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.16.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.17.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.18.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.19.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.20.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.21.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.22.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.23.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.24.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.25.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.26.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.27.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.28.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.29.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.30.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.31.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?127.*) AND (winlog.event_id:\\"22\\" AND QueryName.keyword:* AND QueryStatus:\\"0\\") AND (NOT (QueryResults.keyword:(\\\\(\\\\:\\\\:ffff\\\\:\\\\)?10.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?192.168.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.16.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.17.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.18.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.19.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.20.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.21.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.22.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.23.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.24.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.25.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.26.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.27.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.28.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.29.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.30.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.31.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?127.*))))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30s"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"22\\" AND QueryName.keyword:* AND QueryStatus:\\"0\\" AND QueryResults.keyword:(\\\\(\\\\:\\\\:ffff\\\\:\\\\)?10.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?192.168.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.16.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.17.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.18.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.19.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.20.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.21.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.22.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.23.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.24.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.25.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.26.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.27.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.28.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.29.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.30.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.31.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?127.*) AND (winlog.event_id:\\"22\\" AND QueryName.keyword:* AND QueryStatus:\\"0\\") AND (NOT (QueryResults.keyword:(\\\\(\\\\:\\\\:ffff\\\\:\\\\)?10.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?192.168.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.16.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.17.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.18.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.19.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.20.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.21.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.22.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.23.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.24.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.25.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.26.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.27.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.28.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.29.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.30.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?172.31.* OR \\\\(\\\\:\\\\:ffff\\\\:\\\\)?127.*))))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          },\n          "aggs": {\n            "by": {\n              "terms": {\n                "field": "winlog.ComputerName",\n                "size": 10,\n                "order": {\n                  "_count": "desc"\n                },\n                "min_doc_count": 4\n              },\n              "aggs": {\n                "agg": {\n                  "terms": {\n                    "field": "QueryName",\n                    "size": 10,\n                    "order": {\n                      "_count": "desc"\n                    },\n                    "min_doc_count": 4\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.aggregations.by.buckets.0.agg.buckets.0.doc_count": {\n        "gt": 3\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Possible DNS Rebinding\'",\n        "body": "Hits:\\n{{#aggregations.agg.buckets}}\\n {{key}} {{doc_count}}\\n\\n{{#by.buckets}}\\n-- {{key}} {{doc_count}}\\n{{/by.buckets}}\\n\\n{{/aggregations.agg.buckets}}\\n",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```

```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="22" QueryName="*" QueryStatus="0" (QueryResults="(::ffff:)?10.*" OR QueryResults="(::ffff:)?192.168.*" OR QueryResults="(::ffff:)?172.16.*" OR QueryResults="(::ffff:)?172.17.*" OR QueryResults="(::ffff:)?172.18.*" OR QueryResults="(::ffff:)?172.19.*" OR QueryResults="(::ffff:)?172.20.*" OR QueryResults="(::ffff:)?172.21.*" OR QueryResults="(::ffff:)?172.22.*" OR QueryResults="(::ffff:)?172.23.*" OR QueryResults="(::ffff:)?172.24.*" OR QueryResults="(::ffff:)?172.25.*" OR QueryResults="(::ffff:)?172.26.*" OR QueryResults="(::ffff:)?172.27.*" OR QueryResults="(::ffff:)?172.28.*" OR QueryResults="(::ffff:)?172.29.*" OR QueryResults="(::ffff:)?172.30.*" OR QueryResults="(::ffff:)?172.31.*" OR QueryResults="(::ffff:)?127.*") (EventCode="22" QueryName="*" QueryStatus="0") NOT ((QueryResults="(::ffff:)?10.*" OR QueryResults="(::ffff:)?192.168.*" OR QueryResults="(::ffff:)?172.16.*" OR QueryResults="(::ffff:)?172.17.*" OR QueryResults="(::ffff:)?172.18.*" OR QueryResults="(::ffff:)?172.19.*" OR QueryResults="(::ffff:)?172.20.*" OR QueryResults="(::ffff:)?172.21.*" OR QueryResults="(::ffff:)?172.22.*" OR QueryResults="(::ffff:)?172.23.*" OR QueryResults="(::ffff:)?172.24.*" OR QueryResults="(::ffff:)?172.25.*" OR QueryResults="(::ffff:)?172.26.*" OR QueryResults="(::ffff:)?172.27.*" OR QueryResults="(::ffff:)?172.28.*" OR QueryResults="(::ffff:)?172.29.*" OR QueryResults="(::ffff:)?172.30.*" OR QueryResults="(::ffff:)?172.31.*" OR QueryResults="(::ffff:)?127.*"))) | eventstats dc(QueryName) as val by ComputerName | search val > 3
```


### logpoint
    
```
(event_id="22" QueryName="*" QueryStatus="0" QueryResults IN ["(::ffff:)?10.*", "(::ffff:)?192.168.*", "(::ffff:)?172.16.*", "(::ffff:)?172.17.*", "(::ffff:)?172.18.*", "(::ffff:)?172.19.*", "(::ffff:)?172.20.*", "(::ffff:)?172.21.*", "(::ffff:)?172.22.*", "(::ffff:)?172.23.*", "(::ffff:)?172.24.*", "(::ffff:)?172.25.*", "(::ffff:)?172.26.*", "(::ffff:)?172.27.*", "(::ffff:)?172.28.*", "(::ffff:)?172.29.*", "(::ffff:)?172.30.*", "(::ffff:)?172.31.*", "(::ffff:)?127.*"] (event_id="22" QueryName="*" QueryStatus="0")  -(QueryResults IN ["(::ffff:)?10.*", "(::ffff:)?192.168.*", "(::ffff:)?172.16.*", "(::ffff:)?172.17.*", "(::ffff:)?172.18.*", "(::ffff:)?172.19.*", "(::ffff:)?172.20.*", "(::ffff:)?172.21.*", "(::ffff:)?172.22.*", "(::ffff:)?172.23.*", "(::ffff:)?172.24.*", "(::ffff:)?172.25.*", "(::ffff:)?172.26.*", "(::ffff:)?172.27.*", "(::ffff:)?172.28.*", "(::ffff:)?172.29.*", "(::ffff:)?172.30.*", "(::ffff:)?172.31.*", "(::ffff:)?127.*"])) | chart count(QueryName) as val by ComputerName | search val > 3
```


### grep
    
```
grep -P '^(?:.*(?=.*22)(?=.*.*)(?=.*0)(?=.*(?:.*\\(::ffff:\\)?10\\..*|.*\\(::ffff:\\)?192\\.168\\..*|.*\\(::ffff:\\)?172\\.16\\..*|.*\\(::ffff:\\)?172\\.17\\..*|.*\\(::ffff:\\)?172\\.18\\..*|.*\\(::ffff:\\)?172\\.19\\..*|.*\\(::ffff:\\)?172\\.20\\..*|.*\\(::ffff:\\)?172\\.21\\..*|.*\\(::ffff:\\)?172\\.22\\..*|.*\\(::ffff:\\)?172\\.23\\..*|.*\\(::ffff:\\)?172\\.24\\..*|.*\\(::ffff:\\)?172\\.25\\..*|.*\\(::ffff:\\)?172\\.26\\..*|.*\\(::ffff:\\)?172\\.27\\..*|.*\\(::ffff:\\)?172\\.28\\..*|.*\\(::ffff:\\)?172\\.29\\..*|.*\\(::ffff:\\)?172\\.30\\..*|.*\\(::ffff:\\)?172\\.31\\..*|.*\\(::ffff:\\)?127\\..*))(?=.*(?:.*(?=.*22)(?=.*.*)(?=.*0)))(?=.*(?!.*(?:.*(?=.*(?:.*\\(::ffff:\\)?10\\..*|.*\\(::ffff:\\)?192\\.168\\..*|.*\\(::ffff:\\)?172\\.16\\..*|.*\\(::ffff:\\)?172\\.17\\..*|.*\\(::ffff:\\)?172\\.18\\..*|.*\\(::ffff:\\)?172\\.19\\..*|.*\\(::ffff:\\)?172\\.20\\..*|.*\\(::ffff:\\)?172\\.21\\..*|.*\\(::ffff:\\)?172\\.22\\..*|.*\\(::ffff:\\)?172\\.23\\..*|.*\\(::ffff:\\)?172\\.24\\..*|.*\\(::ffff:\\)?172\\.25\\..*|.*\\(::ffff:\\)?172\\.26\\..*|.*\\(::ffff:\\)?172\\.27\\..*|.*\\(::ffff:\\)?172\\.28\\..*|.*\\(::ffff:\\)?172\\.29\\..*|.*\\(::ffff:\\)?172\\.30\\..*|.*\\(::ffff:\\)?172\\.31\\..*|.*\\(::ffff:\\)?127\\..*))))))'
```



