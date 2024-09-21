# ICMP

Note sul protocollo ICMP.

[**RFC 792**](https://datatracker.ietf.org/doc/html/rfc792)

## Header

```
0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     Type      |     Code      |          Checksum             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             unused                            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |      Internet Header + 64 bits of Original Data Datagram      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Note

ICMP (Protocollo IP 1) non utilizza porte ma **Type** (categoria) e **Code** (dettaglio).

La funzione più famosa è **Ping**:
- Echo-request = type 8, code 0
- Echo-reply = type 0, code 0

Codici Type da conoscere:
- **Type 3** = **Destination unreachable**  
	- Code 0 = Network non raggiungibile  
	- Code 1 = Host non raggiungibile  
	- Code 3 = Porta UDP non raggiungibile  
	- Code 4 = Richiesta frammentazione ma bit DF impostato  
	- Code 9 = Rete proibita dall'amministratore  
	- Code 10 = Host proibito dall'amministratore  

- **Type 5** = **Redirect message** (per informare su rotte migliori)
	- Code 0 = reindirizzamento per network 
	- Code 1 = reindirizzamento per host

- **Type 11** = **Time Exceeded**
	- Code 0 = TTL scaduto
	
	
Messaggio di **HOST UNREACHABLE** è utile per inviare un falso messagio a eventuali port scanner.  

Esempio di sintassi di IPTABLES:  
`iptables <...> -j REJECT --reject-with icmp-host-unreachable`