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

- **Tools**: software utilizzati dall'attaccante come tools per la creazione di allegati malevoli per spear phishing, backdoors verso server C2, tools per trovare password...

- **Tattiche, tecniche e procedure (Tactics, Techniques and Procedures: TTPs)**: step dell'attacco, da recon a compromissione finale/esfiltrazione di dati

<!-- - **Tools**: -->

## SANS Incident Response 6-Steps Pan
- **Prepaation**: viene delineata la policy di sicurezza dell'organizzazione che identifica gli asset sensibili e una strategia di gestione degli incidenti basata sui possibili rischi
- **Identification**: identificazione di possibili incidenti, raccolta di informazioni aggiuntive
- **Containment**: contenimento della minaccia
- **Eradication**: rimozione completa della minaccia
- **Recovery**: ripristino delle macchine compromesse a uno stato normale o comunque il più vicino possibile
- **Lessons learned**: analisi dell'intervento e pianificazione di azioni necessarie per prevenire che incidente si ripeta in futuro

<img src="https://www.forescout.com/wp-content/uploads/2020/01/Forescout-NIST-Framework-The-5-Core-Functions.svg" width="100%" height="auto">

## SOC

3 tipi di SOC:

- **Threat-centric**: ricerca attiva di minacce nella rete
- **Compliance-based**: analizza e configura difesa per rendere l'azienda in linea con i regolamenti di interesse, come PCI DSS o CIS
- **Operation-based**: SOC interno (ComputerSecurityIncidentResponseTeam), continuo assessment degli asset interni

- **SOC interno**: on-site, interamente gestito dall'azienda proprietaria
- **vSOC**: servizio a contratto con terze parti
- **Ibrido**: combinazione dei due sopra