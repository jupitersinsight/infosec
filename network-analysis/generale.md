Microsoft utilizza certificati digitali self-signed che possono generare messaggi di warning quando si analizza il traffico con uno sniffer (esempio in linux).

Quando si cattura del traffico di rete lo sniffer intercetta i dati prima che il CRC sia calcolato per i pacchetti in ingresso (CRC "corrotto") e in uscita.
Tshark genera messaggi di errore mentre altri programmi gestiscono il problema nascondendo direttamente i messaggi di errore.

I messaggi ICMP UNREACHABLE contengono nel payload parte dell'header del messaggio originale e in alcuni casi messaggi per il mittente. Esempio, nel caso di invio di pacchetti con il FLAG DF (Don't Fragment) inserito, nel momento in cui capiti che uno dei router che si trovano lungo il percorso del pacchetto supporti solo una MTU inferiore, ritorna un messaggio ICMP UNREACHABLE che contiene un messaggio nel payload con la dimensione MTU da utilizzare.

Stateful packet inspection: firewall è in grado di determinare quali paccheti possono entrare dall'esterno in quanto generati in risposta a un connessione iniziata dall'interno ma soprattutto è in grado di determinare se un pacchetto con origine diversa è legittimo, e quindi può percorrere la strada del ritorno, in quanto è una risposta attesa da un client. Ad esempio un FTP DATA CHANNEL.


FIN scan (solo con flag FIN impostata) utile per scansione delle porte in Linux. Per RFC quando si riceve un pacchetto con solo FIN flag impostata su una porta aperta non deve esserci risposta perché si tratta di un pacchetto malformato (FIN/ACK è l'unica combinazione ammessa). Mentre quando si riceve su una porta chiusa il Sistema operativo prende in carico la risposta e restituisce un RST/ACK.
Di contro Windows risponde sempre con un RST/ACK rendendo impossibile enumerare le porte aperte con questa tecnica.


Differenza tra fisico e hypervisor, riduzione delle performance a livello di rete in quanto si riducono la Window Size e il Window Scaling.
Per programmatori di hypervisor la gestione dei pacchetti è complicata.
È sconsigliato quindi utilizzare una macchina virtuale dedicata alla raccolta di pacchetti anche se è l'unica ad accedere alla scheda di rete fisica.


TCP Options negoziate da chi inizia e l'altro risponde solo se presentatate.
Tool di packet crafting creano principalmente pacchetti con TCP Options vuote.

In Linux /proc/sys/net/ipv4/xx file modificabili per alterare momentaneamente i valori dell'IP Stack. Per rendere le modifiche permanenti: /etc/sysctl.conf