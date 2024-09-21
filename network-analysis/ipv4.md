# IPv4

Note sull'header del protocollo IPv4.  

[**RFC 791**](https://datatracker.ietf.org/doc/html/rfc791)

## Header

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Note

### IHL o Internet Header Length

Questo campo identifica la lunghezza dell'header in *32 bit words*.  
Condivide il byte con il campo *Version*, quindi è formato da soli 4 bit.  

Il suo valore minimo per un corretto header è 5 (0101), ovvero 20 byte (5*4) mentre il valore massimo è 15 (1111), ovvero 60 byte.

### Identification

Il campo IP ID dell'header IPv4 serve a identificare in maniera univoca i pacchetti IP. 

Il valore univoco assegnato non è gestito allo stesso modo da tutti i sistemi operativi:
 - Windows incrementa di 1 il valore per ogni pacchetto inviato
 - Linux non imcrementa il valore se il pacchetto usa il protocollo ICMP (valore = 0), incrementa il valore su base temporale per UDP mentre lo incrementa in maniera casuale per TCP.

#### Implicazioni

La prevedibilità comporta sempre grandi rischi. In questo caso, un sistema operativo come Windows è il candidato ideale per una tecnica che si chiama **idle scan**. 

Questa tecnica consiste, come già accennato, nello sfruttare un host cui i valori identificativi dei pacchetti incrementino in maniera prevedibile al fine di effettuare un port scan di un secondo host.  

La tecnica prevede l'invio di richieste TCP SYN verso l'host da scansionare usando come IP sorgente l'IP spoofato dell'host con l'IP ID prevedibile.  

Quando l'host vittima riceve la richiesta e la porta è chiusa invia un TCP RST (come da RFC) all'IP spoofato. Questo, ricevendo un RST a una richiesta SYN che non ha inviato, non risponde e quindi siccome non invia pacchetti l'IP ID non incrementa.  

Quando invece l'host vittima riceve la richiesta e la porta è aperta invia un TCP SYN/ACK all'IP spoofato. Questo a sua volta ricevendo un TCP SYN/ACK a una richiesta che non ha iniziato risponde con un TCP RST/ACK e incrementa l'IP ID.  

L'attaccante non deve fare altro che monitorare l'IP ID e registrare quando questo incrementa in corrispondenza di pacchetti con IP spoofato.  
Il target ideale deve quindi avere un IP ID che incrementi in modo prevedibile e deve generare pochissimo traffico.

### FLAGS

Il campo FLAGS è formato dai 3 bit *alti* (`128`, `64`, `32`) e serve a specificare lo stato della frammentazione (no, forse, ultimo frammento, altri frammenti) mentre il campo FRAGMENT OFFSET indica l'offset del frammento e quindi come si inserisce nel Datagram.

#### Implicazioni

La frammentazione dei pacchetti può essere utilizzata per condurre attacchi DoS o per bypassare controlli sul traffico.

Caratteristiche per identificare Fragmentation Attacks (solo l'ultimo frammento dovrebbe essere più piccolo della MTU): lunghezza nell'header IP (byte 2 e 3) inferiore a 512 byte e flag MORE FRAGMENTS attivo (quindi non ultimo frammento) (byte 6 dell'header IP e valore del bit 32 diverso da 0).

Esempio di filtro tcpdump:

`tcpdump -nn -s 1500 -w file.pcap 'ip[2:2]<0x200 and ip[6]&32!=0'`

1) **-nn**: non risolvere nomi  
2) **-s 1500**: packet slicing a 1500 byte  
3) **-w**: scrive nel file specificato  
4) **ip[2:2]<0x200 and ip[6]&32!=0'**: byte da 2 a 3 con valore di 0x200 (512) e bit 32 del byte 6 impostato a 0.

### TTL o Time to Live

Ogni sistema operativo utilizza un time to live differente:
- 32 = Vecchi sistemi Apple
- 64 = Linux, UNIX, e Mac moderni
- 128 = Windows
- 255 = Maggior parte degli apparati di rete

### Implicazioni

Il valore TTL risulta essere un metodo rudimentale per identificare tramite passive fingerprinting il sistema operativo in uso in un host.  
Non si tratta di un valore appunto assoluto e preciso in quanto può essere modificato ma se unito con informazioni di altri protocolli di altri livelli OSI, è sicuramente un dato a cui prestare attenzione.

Uno strumenti che utilizza questo campo per misurare gli hop tra due punti è **Traceroute** che invia pacchetti con TTL crescente (da 1 a salire) fino alla destinazione. Windows utilizza ICMP, Linux è configurabile.

**IPTABLES** permette di filtrare il traffico in base al campo TTL e di scartare, ad esempio, i pacchetti con TTL bassa.  
Il risultato è che traceroute restituisce un valore incompleto, come se ci fosse un buco, che rende la vita ad eventuali attaccanti più difficile.


### Options

Il campo OPTIONS quando utilizzato incrementa la dimensione dell'IP Header (IHL) oltre i 20 byte (valore di minimo consentito e più utilizzato).

Opzioni sfruttate in potenziali abusi sono:
1) Record route - type 7
2) Strict source routing - type 137
3) Loose source routing - type 131

Campi di OPTIONS
```
|copy bit|class (2 bit)|remaining options (bit 5)|lenght 1 byte|pointer 1 byte|data|
```

1) Ogni router che riceve il pacchetto e supporta l'opzione Record Route inserisce il proprio indirizzo IP nel pacchetto. La dimensione del campo "DATA" permette di scrivere fino a 9 indirizzi IP ma uno è quello del mittente, quindi questa opzione può raccogliere fino a 8 indirizzi IP.

2) Definisce una lista di indirizzi IP verso cui il pacchetto deve essere ruotato fino al raggiungimento dell'ottavo IP che identifica la destinazione finale.  
Il primo IP è la sorgente. Affinché funzioni i router devono supportare questa opzione. Ogni volta che il pacchetto entra in un router l'IP di destinazione cambia per combaciare con l'IP successivo specificato nelle IP Options.

3) Gli indirizzi IP specificati nelle IP Options identificano solo i nodi a cui il pacchetto deve arrivare e da li ruotato verso un altro IP nelle Options o verso la destinazione finale. Non è limitato a soli 8 indirizzi IP.  
Se usato per aggirare firewall in uscita, la strada del ritorno NON segue quella del "loose routing" ma il firewall permette lo stesso il passaggio (non sempre e da verificare) perché viene trattata come connessione iniziata dall'interno.