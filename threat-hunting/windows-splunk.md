# Windows

## Splunk + Windows Security + Sysmon

Di seguito una lista di eventi da monitorare, [fonte](https://www.splunk.com/en_us/blog/security/threat-hunting-sysmon-event-codes.html).

[4688: A new process has been created | Windows Security | Malicious Activity](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688)  

Ricerca in Splunk per individuare eventi 4688 **rari**.
```
sourcetype="wineventlog:security"EventCode=4688
| stats count, values(Creator_Process_Name) as Creator_Process_Name by New_Process_Name
| table New_Process_Name, count, Creator_Process_Name
| sort count
```
___
[4738: A user account was changed | Windows Security | Privilege Escalation](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4738)

Ricerca in Splunk per individuare operazioni sugli utenti **2 minuti prima e dopo** la registrazione dell'evento 4738:

```
index=wineventlog
	[search index=wineventlog sourcetype=WinEventLog* EventCode=4738
	| eval earliest=_time-120
	| eval latest=_time+120
	| fields host, earliest, latest]
| table host, sourcetype, EventCode, Message
```
___
[4624: An account was successfully logged on | Windows Security | User compromise](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4624)

Ricerca in Splunk per individuare accessi che non rispettano la normalità:

```
sourcetype="wineventlog:security" EventCode=4624 
 | eventstats avg("_time") as avg stdev("_time") as stdev 
 | eval lowerBound=(avg-stdev*exact(2)), upperBound=(avg+stdev*exact(2))
 | eval isOutlier=if('_time' < lowerBound OR '_time' > upperBound, 1, 0)
 | table _time, isOutlier, body
```
___
[1102: Audit log clearing | Windows Security | Covering the Tracks](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=1102)

Alert **CRITICO** in quanto potrebbe indicare che un TA ha eliminato gli Eventi di Windows nel tentativo di nascondere la propria attività.
___
[1: Process creation | Sysmon](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90001)
___
[22: DNSEvent | Sysmon](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90022)

Ricerca in Splunk per analizzare richiesta DNS da un determinato endpoint specificando nella query SPL il programma utilizzato e il dominio richiesto.

```
source="xmlwineventlog:microsoft-windows-sysmon/operational"  EventCode=22 
EventDescription="DNS Query" host="<hostname>" Image="C:\\Program Files 
(x86)\\Microsoft\\Edge\\Application\\msedge.exe" QueryName="<domain\website>"
```
___
[19: WmiEventFilter activity detected](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90019)  
[20: WmiEventConsumer activity detected](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90020)  
[21: WmiEventConsumerToFilter activity detected](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90021)  

WMI catalogato come Technique in [MITRE ATT&CK Framework](https://attack.mitre.org/techniques/T1047/) data la sua utilità nelle operazioni dei TAs.

