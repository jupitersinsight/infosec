# DNS Hunting

## Fonti e risorse
[Splunk | DNS Detection](https://github.com/rkovar/dns_detection)  
[Splunk | .conf2015](https://conf.splunk.com/files/2015/recordings/2015-splunk-136.mp4?_gl=1*1fnxmxa*_ga*OTM1MDU4Njc5LjE3MTg4MDY3MDY.*_ga_GS7YF8S63Y*MTcxOTUwMTA3NS44LjEuMTcxOTUwMTE4NC42MC4wLjA.*_gcl_au*NzUzMDgwMTYuMTcxOTMwODkwOA..*FPAU*NzUzMDgwMTYuMTcxOTMwODkwOA..*_ga_5EPM2P39FV*MTcxOTQ5OTA5My44LjEuMTcxOTUwMTIxNy4wLjAuMTE3MTM3MzI3)  
[Active Countermeasures | Detecting DNS Backdoors](https://youtu.be/as0isilMYSI?feature=shared)

## DNS Tunneling

Strumenti come *iodine* e *dnscat* sfruttano DNS per creare tunnel e inviare comandi e file o eseguire esfiltrazione.  
Indicatori sono:  
- utilizzo di NULL type query o TXT/MX/CNAME con queries lunghe oltre 200 caratteri
- presenza di alta entropia nelle query o nei nomi di dominio (utile per individuare Malware)
- utilizzo di forme di codifica come base32 e base64

Il logging dei DNS in host, firewall, apparati di rete in generale e strumenti come Zeek (Bro) è fondamentale.  
Così come la creazione di lookup table per il whitelisting di indirizzi IP e nomi di dominio sicuri e attendibili, al fine di ridurre il rumore nelle risposte alle queries nei SIEM.

## DNS Backdoors
Le comunicazioni delle backdoor che sfruttano il sistema dei DNS potrebbero non risultare da un'analisi degli eventi su base temporale, ovvero l'heartbeat prodotto da un grafico *numero eventi*x*intervallo di tempo* potrebbe non mostrare eventi fuori dal normale.  

Questo significa che il protocollo è usato da molti apparati, DNS infatti è essenziale, oppure che non c'è regolarità nelle comunicazioni.  
Analizzare invece il *numero di eventi*x*sottodomini contattati* può far saltare all'occhio quali domini risultano nella maggior parte delle richieste (troncando al secondo/terzo sottodominio o comunque la porzione fissa e non variabile).

## C2 over DNS
----------------------------
L'agente invia una richiesta DNS per un nome di dominio formato, ad esempio, così: *1s1d6fef1aef5sdf98e.evil.com*.  

La richiesta di risoluzione DNS arriva al local resolver che non avendo informazioni a riguardo si rivolge al server più in alto nella catena che a sua volta potrebbe reindirizzare la richiesta fino a che si arriva al server DNS autoritativo per il TLD *.com*.  

Questo server sa che evil.com è gestito da un (o più) server autoritativo(i): significa che l'attaccante quando ha comprato il dominio evil.com ha indicato come server autorevole il server C2.  

La risposta del server viene quindi impacchettata e inviata all'host dove risiede l'agente. Questa risposta contiene la risposta alla query dell'agente: *dormi* o *esegui questo comando*.  

In ogni caso la risposta anche un eventuale comando (quindi l'output) è codificato e inviato al server C2 seguendo la stessa prassi.  

Questa tecnica genera un valore facilmente individuabile: **troppi FQDNs**.  

Nella media la quantità di FDQNs (domini e sottodomini) appartenenti a una società è inferiore 10, nel caso di aziende come Microsoft, Akamai, Google, Amazon... il dato si attesta tra i 200 e i 600.  

Un valore superiore a 1000 **deve** destare sospetti. 

Esempio: sommando il numero di FQDNs univoci (non il numero di richieste e risposte) di evil.com ci si accorge del traffico C2 perché il totale supera il 1000.  

