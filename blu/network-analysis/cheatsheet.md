**Filtra file PCAP per identificare User-Agent univoci in richieste HTTP**  
Ad esempio in caso di scansione con script o tool come gobuster, sqlmap...

```cmd
tshark -r esempio.pcap -Y http.request -T fields -e http.host -e http.user_agent | sort.exe /unique
```

**Filtra file PCAP per identificare comandi inviati tramite webshell**  

```cmd
tshark -r esempio.pcap -Y "http.request.uri contains \"<parametro specificato nella webshell>\"" -T fields -e http.host -e http.user_agent -e http.request.uri -2 
```

**Filtra file PCAP per identificare richieste POST univoche**

```cmd
tshark -r esempio.pcap -Y http.request.method==POST -T fields -e http.host -e http.user_agent | sort.exe /unique
```
