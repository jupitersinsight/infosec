# DNS Hunting

## Fonti e risorse
[https://github.com/rkovar/dns_detection](https://github.com/rkovar/dns_detection)
[.conf2015](https://conf.splunk.com/files/2015/recordings/2015-splunk-136.mp4?_gl=1*1fnxmxa*_ga*OTM1MDU4Njc5LjE3MTg4MDY3MDY.*_ga_GS7YF8S63Y*MTcxOTUwMTA3NS44LjEuMTcxOTUwMTE4NC42MC4wLjA.*_gcl_au*NzUzMDgwMTYuMTcxOTMwODkwOA..*FPAU*NzUzMDgwMTYuMTcxOTMwODkwOA..*_ga_5EPM2P39FV*MTcxOTQ5OTA5My44LjEuMTcxOTUwMTIxNy4wLjAuMTE3MTM3MzI3)

## DNS Tunneling

Strumenti come *iodine* e *dnscat* sfruttano DNS per creare tunnel e inviare comandi e file o eseguire esfiltrazione.  
Indicatori sono:  
- utilizzo di NULL type query o TXT/MX/CNAME con queries lunghe oltre 200 caratteri
- presenza di alta entropia nelle query o nei nomi di dominio (utile per individuare Malware)
- utilizzo di forme di codifica come base32 e base64

Il logging dei DNS in host, firewall, apparati di rete in generale e strumenti come Zeek (Bro) è fondamentale.  
Così come la creazione di lookup table per il whitelisting di indirizzi IP e nomi di dominio sicuri e attendibili, al fine di ridurre il rumore nelle risposte alle queries nei SIEM.
