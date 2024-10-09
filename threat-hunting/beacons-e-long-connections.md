# Beacon analysis

## Fonti e risorse

[**Threat Hunting Beacon Analysis**](https://youtu.be/FzGbVMntLT0?feature=shared)


## Cosa significa *Beacon*?

I beacons sono chiamate inizializzate da un client o agent verso un determinato sistema esterno, come un malware che cerca di contattare ripetutamente il server C2.

## Cosa significa *Long Connection*?

Le long connections sono connessioni che risultano attive da molto tempo, nell'ordine di ore o giorni. Sono d'interesse perché una sessione mantenuta attiva per molto tempo indica che lo scambio tra le parti è continuo o che una delle due parti cerca di mantenere il canale aperto, questo al fine di attivare il trasferimento o la ricezione di dati.

# Come individuare i beacons e le long connections nei file di *Packet Captures* o nei log di *Zeek*?

<details>
    <summary><b>Zeek vs PCAPS</b></summary>

I file .pcap memorizzano l'intero binario dei pacchetti, quindi a dimensione intera e possono raggiungere dimensioni notevoli in poco tempo.

Di contro Zeek è un *network recorder* e memorizza in testo il contenuto della sessione tra due parti, questo permette una raccolta dati che, a parità di dimensione con un file pcap, spazia su un arco temporale maggiore.
</details>
<br>

Prima di tutto è necessaria una base dati corposa, almeno 12-24 ore, per avere maggiori informazioni da analizzare e *potenzialmente* maggiori possibilità di individuare del traffico malevolo.  

Successivamente è utile identificare le conversazioni tra IP al fine di *ripulire* velocemente i risultati.  

<details>
    <summary><b>Cos'è il safelisting?</b></summary>

Marcare come sicure sessioni legati a servizi fidati e necessari all'attività lavorativa.  

Si tratta di revisionare i dati raccolti da Zeek piuttosto che file pcap o altre risorse utilizzate per registrare il traffico, identificare il traffico da considerare sempre sicuro e ottimizzare gli strumenti.  

Questa attività aiuta ad eliminare il rumore dai dati raccolti in modo che, ripetuto e applicato ogni volta, renda più semplice l'identificazione del solo traffico malevolo.
</details>  

## Beacons
Essendo i **beacons** comunicazioni ricorrenti più o meno regolari nel tempo e dalla dimensione fissa nella lunghezza delle richieste i valori da tenere in considerazione sono:
- connessioni persistenti nell'arco di almeno 24 ore
- regolarità nella persistenza (richieste ogni XX minuti)
- jitter (connessioni legittime non hanno bisogno di introdurre jitter)
- eventuali risoluzioni DNS (nomi strani) o accessi diretti a IP pubblici
 
Gli sviluppatori di malware infatti, o in ogni caso gli attori malevoli, possono introdurre tecniche per rendere **meno** identificabili i loro agent:
- *comunicazioni irregolari*: introduzione di latenza nelle richieste e quindi si *rompe* la regolarità ma l'analisi del traffico in un arco temporale lungo permette comunque l'identificazione dei beacons
- *dimensione richiesta e contenuto irregolare*: introduzione di forme di codifica o crittografia per nascondere il contenuto delle comunicazioni o l'inserimento di byte di padding per celare ulteriormente il contenuto. In ogni caso, l'appesantimento del contenuto fa si che gli host infetti risultano tra i dispositivi più *loquaci*

### Tecniche di analisi
----------------------------
Esistono diversi modi per identificare traffico C2 (**i migliori risultati si ottengono dalla combinazione di tutte le tecniche**)

Combinando le due tecniche è possibile tracciare un grafico che identifica la frequenza, il peso delle sessioni ma anche l'attivazione e la fase di *actions-on-objective* senza avere a disposizione il payload (che potrebbe essere crittografato perché all'interno di un tunnel TLS).  

### Durata e frequenza della sessione
----------------------------
Al fine di identificare sessioni di traffico C2 utilizzando il tempo come variabile occorre avere dati che coprano un arco temporale di 12 o 24 ore, analizzare quindi quali sessioni (domanda e risposta) si presentano con una frequenza fissa o comunque con variazioni di qualche secondo, *jitter*.

### Dimensione della sessione
----------------------------
Utilizzando la dimensione delle sessioni come variabile è possibile individuare anche l'eventuale attivazione del malware e quindi quando il canale C2 da inattivo diventa attivo.  
Generalmente l'agente *dormiente* contatta il server C2 per richiedere istruzioni (dimensione del pacchetto fissa in quanto il contenuto della richiesta non varia) e la risposta è negativa (anche qui la dimensione è fissa in quanto il contenuto della risposta non varia). Questo **schema** prende il nome di *heartbeat*.  
Quando alla richiesta dell'agente corrisponde invece un comando la dimensione della risposta sarà diversa così come il traffico generato successivamente dall'agente che, per inviare il risultato dell'operazione effettuata, genererà pacchetti con dimensione diversa.  

Valutando il peso delle sessioni è possibile anche identificare se c'è stata *exfiltration* e la sua portata.  

<details>
    <summary>Esempi</summary>

La creazione di grafici sulla base dei dati raccolti che tenga in considerazione la quantità di eventi (asse x) e del tempo (asse y) a intervalli di 1 ora, permette di tracciare una linea immaginaria che risulta essere la costante nelle comunicazioni.
<br>
Un altro grafico utile prende in considerazione invece la latenza tra le comunicazioni di una specifica sessione (o comunque dialogo tra due parti precise) che varierà di pochi secondi.
</details>

### Quando non funziona
----------------------------
Chiaramente questa tecnica non funziona in assenza di comunicazioni C2.  
Malware rari ma distruttivi come è stato *NotPetya*: utilizzo dell'exploit *EternalBlue* per la massima diffusione nei sistemi non coperti da patch, cancellazione definitiva dei file e crittografazione dell'intero disco, finto indirizzo Bitcoin per mascherare l'attacco e le intenzioni.  
Essendo un malware creato e distribuito con il solo intento di fare il maggiore danno possibile, è già armato di tutto quanto serve allo scopo senza che attenda istruzioni dal centro di controllo.

## Long connections
Come anticipato le long connections sono connessioni mantenute attive per molto tempo.  

In questo caso l'analisi avviene monitorando il tempo totale delle sessioni salvo poi muoversi verso l'identificazione delle due parti (ovvero se l'IP esterno è utilizzato da un attore malevolo).

<details>
    <summary><b>Problema Zeek</b></summary>

**! Attenzione !**: Se si utilizza Zeek per registrare la sessione... questa non sarà registrata fino a quando non sarà chiusa.  
Purtroppo Zeek scrive su log le connessioni/sessioni solo una volta chiuse. Questo significa che se esiste una sessione aperta da 20 giorni, Zeek ne scriverà nei log e ne darà notifica solo quando questa verrà chiusa, potrebbe essere al giorno 21 o 70... chi lo sa.  

**! Workaound !**: scrivere un tool che interroghi Zeek per vedere se ci sono file da scrivere in un log e salvarlo quindi in un file dedicato in una repository dedicata. 
</details>
<br>
<details>
    <summary><b>Come identificare il traffico legittimo</b></summary>

Se il FQDN non aiuta bisogna ricorrere ad altre risorse per chiarire la natura della destinazione:
- risalire all'ASN, chi ne è proprietario
- geolocalizzare la destinazione
- IP Delegation (Reverse DNS)
- utilizzare strumenti come VirusTotal, AbuseIPDB...
</details>