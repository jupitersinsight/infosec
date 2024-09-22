# TCP

Note sul protocollo TCP.

[**RFC 9293**](https://datatracker.ietf.org/doc/html/rfc9293)

## Header


```
	0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |       |N|C|E|U|A|P|R|S|F|                             |
   | Offset| Rsrvd |S|W|C|R|C|S|S|Y|I|           Window            |
   |       |       | |R|E|G|K|H|T|N|N|                             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                           [Options]                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               :
   :                             Data                              :
   :                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   ```
   

## Note

TCP supporta solo comunicazioni unicast (per 3-way handshake).  

L'header ha una dimensione minima di 20 byte (che può crescere fino a 60 byte) ma è raro in quanto le Options sono spesso utilizzate.  

Al contrario di quanto succede per il protocollo IP, il campo Options vuoto in SYN potrebbe essere un **indicatore di packet crafting**.

### Sequence number e Acknowledgment number

**Sequence Number** e **Acknowledgment Number** campi entrambi di 4 byte l'uno.  

Lo scopo principale del Sequence Number è di individuare eventuali comunicazioni mancanti così come l'Acknowledgment Number.  
Inoltre può aiutare nell'identificatore quanti dati sono stati trasmessi nella sessione senza tenere traccia di ogni singolo pacchetto.

Il valore iniziale del Sequence Number dovrebbe essere casuale e poi incrementato:
- di 1 se il flag SYN è = 1 (una volta per handshake)
- della dimensione di dati nel payload 

L'Acknowledgment Number conferma la ricezione del dato calcolando il numero del Sequence Number successivo.

Il flusso viaggia in entrambe le direzioni simultaneamente ed è per questo più semplice analizzarlo come due flussi separati, uno per volta.

Esempio:

```
> Seq. = 150
> Data = 100 bytes
> ACK 1000
					< Seq. = 1000
					< Data = 50 bytes
					< ACK 250
> Seq. = 250
> Data = 25 bytes
> ACK 1050
					< Seq. = 1050
					< Data = 125 bytes
					< ACK 275
> Seq. = 275
> Data = 125 bytes
> ACK 1175
					< Seq. = 1175
					< Data = 61 bytes
					< ACK 400
> Seq. = 400
> Data = 37 bytes
> ACK 1236
					< Seq. = 1236
					< Data = 50 bytes
					< ACK 437
...
```

I Sequence number assoluti sono quelli iniziali (RAW) che si trovano nel range 0 - 4.294.967.295.   

I numeri di sequenza relativi semplificano il lavoro in quanto si considera il valore di partenza uguale a "0" per poi sommare e calcolare il Sequence Number e Acknowledgment Number.

### Data Offset

Il campo Data Offset è come il campo Length dell'header IP.

### Control bits

I 6 low-order bit del byte 13 sono:
- FIN, bit 1 = termina correttamente la sessione
- SYN, bit 2 = inizializza la sessione
- RST, bit 4 = indica una porta chiusa o la connessione si chiude a causa di un errore
- PSH, bit 8 = indica alla destinazione di inviare i dati dal buffer (8096 byte) all'applicazione in ascolto (il buffer evita una condizione stile DoS)
- ACK, bit 16 = non attivo solo nel primo pacchetto della sessione e quando una porta aperta riceve un ACK inaspettato (risposta solo con RST)
- URG, bit 32 = utilizzato per ignorare il buffer (TCP è FIFO) e indicare alla destinazione di processare il pacchetto e i dati la cui fine è indicata dal campo Urgent Pointer (esempio, CTRL+C in SSH)

**Half Open e Half Closed state**  

Half open significa che una delle due parti ha terminato la sessione senza informare l'altra parte oppure se le due parti sono desincronizzate per problemi dovute alla rete o al riavvio di una delle due parti.

Half closed significa che una delle due parti ha inviato un FIN/ACK ma l'altra parte ha ancora dati da inviare e quindi, fino a che il trasferimento non è concluso, invia un ACK ma non un FIN/ACK.  

Esempio: durante il download di un file il client, dopo aver stabilito una established connection (3 way) e inviato la richiesta GET (HTTP), invia un FIN/ACK. Il server però non ha ancora inviato il file o non ha ancora terminato il trasferimento.  
Invia quindi un ACK come Acknowledgment della richiesta del client di terminare la sessione ma continua a trasmettere i dati.  
Alla fine, il server invia un FIN/ACK a cui corrisponde un Acknowledgment del client.

**Bit Masking** 

Il Bit Masking consiste in una sintassi speciale per accedere al valore dei bit specificati.  

Sintassi: `Header [byte] & bit operator value`

Esempio:

- `'tcp[13]&2!=0'`: match quando SYN flag impostata
- `'tcp[13]&1!=0'`: match quando FIN flag impostata
- `'tcp[13]&8=8':` match quando PSH flag impostata

- `'tcp[13]&18=18':` match quando SYN/ACK impostata
- `'tcp[13]&17=17'`: match quando FIN/ACK impostata
- `'tcp[13]&17!=0'`: match quando FIN e/o ACK flag impostata

- `'ip[6]&32=32'`: match quando MoreFrag flag impostata
- `'ip[0]&15>5'`: match quando IHL > 20 bytes / bit 15 non esiste ma in questo caso controlla i bit 1,2,4,8 (15)

Esempio con tcpdump: `tcpdump -nn -r ssh.pcap 'tcp[13]&3!=0`

### Window

Il campo Window nei byte 14 e byte 15 serve lo scopo di avere Flow Control e identifica la quantità di dati (non pacchetti) che una parte indica all'altra che può ricevere prima di dover notificare con un ACK la ricezione e accettare poi altri dati.  

La dimensione della Window Size può crescere e diminuire durante una sessione.  

Essendo il campo di 2 bytes il valore massimo della Window Size è pari a 65535 bytes.

Window Size = 0 è un valore valido e significa *attendi*.  

Per mantenere la sessione attiva il client può inviare ACK a cui risponde il server sempre con Window Size = 0.

### Checksum

Il campo Checksum  è formato dai Bytes 16 e 17 dell'header.  
Verifica la coerenza della trasmissione ed effettua il controllo sui campi:
- IP sorgente (bytes 12-15 dell'header IP)
- IP destinazione (bytes 16-19 dell'header IP)
- Numero di protocollo (byte 9 dell'header IP)
- Header TCP
- Payload

### Urgent pointer

Urgent Pointer formato dai Bytes 18 e 19  
Identifica l'ultimo byte dei dati urgenti, bypassa la normale coda FIFO.

## Options

Le TCP Options sono generalmente passate in pacchetti con flag SYN o SYN/ACK.  

Le Options sono negoziate da chi inizia la conversazione. L'altra parte non può proporre l'utilizzo di Options non specificate dal client ma solo confermare di supportarle o meno, tutte o in parte.  

**I tool di packet crafting generalmente creano pacchetti con TCP Options vuote, quindi identificabili.**

Essendo in formato 32-bit (4 byte) devono essere divisibili per 4 quindi se il totale delle opzioni utilizzate è 18 byte, si aggiungeranno 2 byte di padding.  

Struttura delle Options: KIND e LENGTH (tipo e dimensione dell'opzione).

```
_______________________________________________________________________________________
| KIND | LENGTH | NAME          | SYN | SHORT DESCRIPTION		                      |
|------|--------|---------------|-----|-----------------------------------------------|
|   0  |    1   | End of options|     |                         					  |
|------|--------|---------------|-----|-----------------------------------------------|
|   1  |    1   | No operations |     |  Padding per allineamento 					  |
|------|--------|---------------|-----|-----------------------------------------------|
|   2  |    4   | MSS           |  X  | MTU per segment locali   					  |
|------|--------|---------------|-----|-----------------------------------------------|
|   3  |    3   | Window scale  |  X  | Moltiplicatore per WSize  					  |
|------|--------|---------------|-----|-----------------------------------------------|
|   4  |    2   | Selective ACKS|  X  | Ripristino avanzato da errori di trasmissione |
|------|--------|---------------|-----|-----------------------------------------------|
|   8  |    10  | Timestamp     |     | Per determinare ordine di invio dei pacchetti |
|------|--------|---------------|-----|-----------------------------------------------|
```
**MSS**  

La Maximum Segment Size (MSS) identifica la dimensione massima che un segmento può avere all'interno della rete locale ed è calcolato rimuovendo l'header del frame, il frame check sequence, l'header IP e l'header TCP.

Esempio:

Pacchetto IP senza VLAN: 1518 byte
Frame header:			 - 14 byte
Frame Check Sequence:    - 4 byte
IP header minimo:        - 20 byte
TCP header minimo:       - 20 byte
						------------
MTU:     				 1460 byte (valore più comune)

Durante l'handshake questo valore rappresenta la base che moltiplicata di almeno dieci volte corrisponde alla Window Size.

La dimensione massima del campo Window Size è 65535 che al giorno d'oggi è poco.  

**Window Scaling**  

L'Option Window Scaling è utilizzata per moltiplicare il valore della Window Size e ottenere la dimensione della **vera Window Size** (non come campo ma come dimensione della finestra da cui far passare i dati)(come i sequence number o acknowledgment number, anche la window size è per direzione).  

L'IP Stack di ogni SO ha valori preconfigurati per determinare la quantità massima iniziale di dati che può accettare. 

Esempio.  
Window Size = 64.240 bytes  
Window Scale = 8

Vera Window Size = 64.240 x 2^8 (256) = 16.445.440 bytes 

**Selective acknowledgments**  

L'opzione SACK permette alle due parti di identificare i pezzi di comunicazione andati perduti e ritrasmettere solo quelli (per poi riordinarle correttamente).  

Inizialmente una comunicazione TCP si bloccava fino a che la parte mancante non arrivava per poi riprendere da quel punto lo scambio dei dati.

**Timestamp**  

Non identifica il momento temporale d'invio del pacchetto ma se il Sequence number ritorna a 0, dopo che il suo valore ha raggiunto il tetto massimo, il valore del campo Timestamp, che è inizializzato in maniera randomica, aiuta a riordinare correttamente i pacchetti.

# Tip per scansione

FIN scan (segmento con solo flag FIN impostata) utile per scansione delle porte di un sistema Linux in quanto applica quanto definito dalla RFC.  

Secondo la RFC, infatti, quando un sistema riceve un pacchetto con solo FIN flag impostata su una porta aperta **non** deve esserci risposta in quanto si tratta di un pacchetto malformato (FIN/ACK è l'unica combinazione ammessa).  

Mentre quando riceve il pacchetto su una porta chiusa il sistema operativo prende in carico la risposta e restituisce un **RST/ACK**.

In questo modo è possibile individuare quali porte sono chiuse e quali sono aperte.

Di contro Windows risponde sempre con un RST/ACK e questo rende impossibile, usando questa tecnica, enumerare le porte aperte.