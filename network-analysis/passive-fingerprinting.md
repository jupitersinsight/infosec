ttl  



**icmp sequence number**  
La dimensione minima dell'header è di 4 byte.  
I primi 4 byte sono sempre uguali per ogni Type mentre i secondi 4 byte variano in base al Type e code.  

Per esempio nell'Echo all'header (PING) si aggiungono un campo di due byte definito come **Identifier** e un secondo campo di due byte definito **Sequence number**, il cui scopo è di tenere traccia delle Echo request inviate e reply ricevute.  

Solo la sorgente aggiorna questi campi se necessario. La destinazione semplicemente copia e risponde.
Big Endian e Little Endian inclusi ma ormai si fa riferimento solo ai valori Big Endian.

Windows non implementa alla lettera le indicazioni della RFC e non resetta il Sequence Number (Identifier rimane 1) e rimane incrementale nel tempo. Solo un riavvio lo reimposta. Il problema è che essendo un indicatore che non si resetta mai potrebbe indicare il tempo online del sistema. Se ad esempio è stata rilasciata una patch urgente contro una 0 day, il valore della seguence number può indicare se il sistema è stato riavviato (e quindi probabilmente protetto se la patch richiede un riavvio) oppure vulnerabile [Recon phase].

Linux per contro mantiene sequenziale il Sequence Number (1,2,3..) e lo resetta. L'identifier è basato sul tempo e quindi molto meno prevedibile.
Messaggio in ICMP è diverso da SO a SO = fingerprinting
Payload è definito dall sorgente e rimane uguale nelle risposte anche se cambia il SO.


**tcp**

Anche le Options possono essere utilizzate per passive fingerprinting in quanto possono essere presentate in ordine diverso, dipende dal SO.


