## Microsoft, i suoi certificati self-signed e messaggi di errore degli sniffer

Microsoft utilizza certificati digitali self-signed che possono generare messaggi di warning quando si analizza il traffico con uno sniffer (esempio in linux).

## Sniffer e il problema dell'errore CRC

Quando si cattura del traffico di rete lo sniffer intercetta i dati prima che il CRC sia calcolato per i pacchetti in ingresso (CRC "corrotto") e in uscita.  

Tshark genera messaggi di errore mentre altri programmi gestiscono il problema nascondendo direttamente i messaggi di errore.

## Stateful packet inspection

Stateful packet inspection: firewall è in grado di determinare quali paccheti possono entrare dall'esterno in quanto generati in risposta a un connessione iniziata dall'interno ma soprattutto è in grado di determinare se un pacchetto con origine diversa è legittimo, e quindi può percorrere la strada del ritorno, in quanto è una risposta attesa da un client. Ad esempio un FTP DATA CHANNEL.

## Differenze di Window Size e Window Scaling tra host e hypervisor

Una delle differenze tra stack IP fisico e quello virtuale di un hypervisor sta nella riduzione dei valori massimi di Window Size e Window Scaling.  

Per questo motivo è sconsigliato utilizzare una macchina virtuale dedicata alla raccolta di pacchetti, anche se avesse a completa disposizione una scheda di rete fisica.

## Linux e il potere di alterare (temporaneamente) i valori dello stack IP

In Linux i file al percorso `/proc/sys/net/ipv4/*` contengono valori dello stack IP che sono modificabili e subito aggiornati. 

Per rendere le modifiche permanenti le stesse vanno riportate nel file `/etc/sysctl.conf`.