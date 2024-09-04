## Honeyports

Apertura porte con servizi inutili che non portano a nulla o utilizzo di tools come [**portspoof**](https://github.com/drk1wi/portspoof) che restituiscono valori falsificati in caso di scansione.  
**Portspoof** risponde alle scansioni con pacchetti SYN+ACK, tutte le porte risultano aperte e allo scanner sono inviate informazioni falsificate ovvero banner di servizi *pescati* a caso da un database.

## Honeydocs

Creazione di documenti *esca* che se aperti fanno scattare alert.

## Honeyusers

Creazione di utenti *esca* per intercettare tentativi di autenticazione tramite alert mirati.

## Canarytokens

[**canarytokens.org/nest]**(https://canarytokens.org/nest/)