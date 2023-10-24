VirusTotal: https://www.virustotal.com/gui/home/upload


## Piramide del dolore.

La Piramide identifica 6 indicatori di compromissione da parte degli attaccanti: più si sale verso il vertice più sarà _doloroso_ per gli attaccanti trovarsi la strada sbarrata essendo costretti a ripianificare l'attacco, a ripetere alcuni step o a desistetere dal proseguire. 

<img src="https://4.bp.blogspot.com/-EDLbyYipz_E/UtnWN7fdGcI/AAAAAAAANno/b4UX5wjNdh0/s1600/Pyramid+of+Pain+v2.png" width="100%" height="auto" alt="Piramide del dolore"></img>

- **Hash values**: Il primo step, più semplice, è l'identificazione di attività tramite la ricerca di hash note (signature).  

- **Indirizzi IP**: Il secondo step è la ricerca di indirizzi IP connessi a domini/operatori malevoli e quindi bloccarli.  

- **Nomi di dominio**: Il terzo invece è il blocco di domini che risultano utilizzati in attacchi.

**Questi primi 3 livelli sono i meno _dolorosi_ e i più facili per gli attaccanti da riorganizzare**

- **Artefatti di rete/host**: Il quarto livello include la raccolta di artefatti di rete o lasciati negli host. Per artefatti si intendono:
    - pattern URI
    - informazioni di C2 racchiuse nei protocolli di rete
    - stringhe di User-Agent diverse dal normale
    - [...]
    - chiavi in valori di registro
    - servizi malevoli
    - file e cartelle fuori dal normale

<!-- - **Tools**: -->