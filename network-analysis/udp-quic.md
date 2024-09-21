# UDP

Note sul protocollo UDP e QUIC.

[**RFC 768**](https://datatracker.ietf.org/doc/html/rfc768)

## Header

```

				  0      7 8     15 16    23 24    31
                 +--------+--------+--------+--------+
                 |     Source      |   Destination   |
                 |      Port       |      Port       |
                 +--------+--------+--------+--------+
                 |                 |                 |
                 |     Length      |    Checksum     |
                 +--------+--------+--------+--------+
                 |									 |
                 |          data octets 			 |
                 +-----------------------------------+
```

## Note 

La dimensione dell'header è fissa a 8 byte.

Questo protocollo è stato *bistrattato* per molto tempo in quanto non ha in sè alcun controllo sulla ricezione (e in corretto ordine) dei dati. 

Con le tecnologie odierne e la necessità di ridurre l'overhead dovuto al costo computazionale di numerosi sessioni TCP, grandi aziende come Google e Facebook hanno riscoperto la leggerezza dell'header del protocollo UDP.  

Su di esso hanno appoggiato un nuovo protocollo, QUIC, che ottimizza gli handshake propri dei protocolli usati maggiormente nelle sessioni web, ovvero TCP-TLS-HTTP.

**!! QUIC non sostituisce UDP**.

Da punto di vista della sicurezza si tratta di un protocollo relativamente nuovo e attivamente revisionato e modificato. Di fondo però rimane un piccolo problema, ovvero chei in nome della velocità qualcosa è stato sacrificato: questo protocollo infatti è in grado salvare in una cache info sulle sessioni al fine di velocizzare le comunicazioni.

QUIC permette la trasmissione di più informazioni allo stesso tempo, ogni "sessione" ha un numero identificativo univoco e se qualche pacchetto non arriva a destinazione, questo impatta solo quella "sessione".  
Il protocollo è in grado di ripristinare la "sessione" e recuperare gli eventuali pacchetti persi.

Le comunicazioni non sono inizializzate solo dai client ma una volta che il "collegamento" è stabilito anche il server può iniziare a trasmettere dati non richiesti.  

Queste sessioni, o *stream*, possono anche durare per **molto** tempo fino a un reset manuale.
