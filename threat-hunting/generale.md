## Cos'è il Threat Hunting  
----------------------------
Il Threat Hunting indica la ricerca preventiva di indicatori di attività malevola.  

Trigger per Threat Hunting sono tempo (ogni giorno, in occasione di un evento, un giorno prestabilito...) e processo (dopo aver fatto altre attività si fa TH...) ma non gli alert perché in quel caso si è già in Incident Response.  

Analizzando prima la rete è possibile individuare traffico malevolo per poi analizzare nel dettaglio passando a un SIEM i log di eventuali host. Successivamente muoversi nella fase di Incident Response.  

Il Threat Hunting è passivo per natura. Analizzare pacchetti da file PCAP o log di Zeek, interrogare un SIEM e altre attività passive non lascia tracce che l'attaccante può interpretare come attività di analisi e che quindi qualcuno l'abbia scoperto.  

Ogni azione attiva non ponderata rischia di interrompere sì l'attacco ma di perdere molte tracce che l'attaccante potrebbe lasciare dietro di sé e non avere quindi sufficienti dati di Threat Intelligence per poterlo "catalogare".

## Tools, metodo e tecniche minori
----------------------------
Utilizzando come base la VM scaricabile da activecountermeasures, utilizzare il comando di Zeek per leggere il file PCAP e generare i log di Zeek: `zeek readpcap <absolute path of the source> <absolute path of the destination>`.  

Importare poi i log in RITA: `rita import -l <absolute path to the folder with the logs> -d <specify a dataser name>`.  

Per verificare i dataset disponibili: `rita list`.  

Per caricare una vista con un dataset: `rita view <dataset name>`.  

Estrarre informazioni su IP o Dominio da log di Zeek per un controllo rapido: `fq <ip or domain name>`

Approfondire elementi sospetti utilizzando strumenti come whois, tcpdump, tshark, dig, tool personalizzati, strumenti e ricerca online.



### Mimetype Mismatch
----------------------------
Il server C2 invia un file, ad esempio una immagine .jpg, a un client ma nell'header HTTP specifica di trattarlo come un file di testo. Tipo di file e uso non combaciano e questo significa che quel file non è una immagine ma più probabilmente una lista di comandi da eseguire.

### HTTP User Agent "rari"
----------------------------
User agent rari nel senso di strani/univoci possono identificarsi con traffico malevolo

### Connessioni a host IP
----------------------------
La connessione non avviene verso un FQDN ma direttamente a un indirizzo IP.


### Porta:Applicazione non combaciano
----------------------------
Molti protocolli *standard* hanno porte assegnate e riconosciute, come ad esempio la ports 80 per HTTP e 443 per HTTPS.  
Questo non impedisce a nessuno di utilizzare queste porte per altri scopi, ad esempio utilizzare SSH non sulla porta 22 ma sulla porta 443.  
Il problema è che molti programmi di packet capture (anche i firewall) si basano sull'associazione standard e in automatico *marcano* il traffico in base a quello: *qualunque sessione verso la porta 443 la etichetto come HTTPS*.  

Altri tool, come Zeek (e questo mi torna utile per i miei script), **analizzano** il livello 7 (Application) e sono in grado di identificare se si tratta, ad esempio, di traffico HTTPS sulla porta 443 o se è invece traffico di altro tipo.  



### HTTP User Agent e JA3
----------------------------
User agent utilizzati in HTTP possono tradire un host compromesso e rivelare che qualcosa non quadra perché sono *sbagliati* sintatticamente o per quell'host: esempio, un HTTP-UA di una versione di Windows che **non** corrisponde alla versione installa nell'host.  

Le hash JA3 sono delle hash dei SSL Hello dei client inviate nelle prime fasi di una negoziazione SSL o TLS. Utilizzate anche queste come signature per identificare evetuali malware noti o *novità* nella rete in quanto nuovi (queste hash sono univoche e difficilmente cambiano).  

### TLS 1.3
----------------------------
TLS cripta lo SNI e potrebbe non essere visibile.  
Funziona con query A e poi HTTPS dal client al server per richiedere un tunnel criptato.


### Telemetria
----------------------------
Sysmon: Eventi ID 3 - connessioni di rete da applicazioni quindi identificare *traffico:app*