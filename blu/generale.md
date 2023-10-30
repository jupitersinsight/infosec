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

## SANS Incident Response 6 Steps Plan
- **Prepaeation**: viene delineata la policy di sicurezza dell'organizzazione che identifica gli asset sensibili e una strategia di gestione degli incidenti basata sui possibili rischi
- **Identification**: identificazione di possibili incidenti, raccolta di informazioni aggiuntive
- **Containment**: contenimento della minaccia
- **Eradication**: rimozione completa della minaccia
- **Recovery**: ripristino delle macchine compromesse a uno stato normale o comunque il più vicino possibile
- **Lessons learned**: analisi dell'intervento e pianificazione di azioni necessarie per prevenire che incidente si ripeta in futuro


## SOC

**3 tipi di SOC:**

- **Threat-centric**: ricerca attiva di minacce nella rete
- **Compliance-based**: analizza e configura difesa per rendere l'azienda in linea con i regolamenti di interesse, come PCI DSS o CIS
- **Operation-based**: SOC interno (ComputerSecurityIncidentResponseTeam), continuo assessment degli asset interni
----
- **SOC interno**: on-site, interamente gestito dall'azienda proprietaria
- **vSOC**: servizio a contratto con terze parti
- **Ibrido**: combinazione dei due sopra

**Dati per analisi**:

- **Session Data**: rappresenta un sunto della comunicazione tra due endpoint in cui è solo possibile vedere IP sorgente/destinazione, porta sorgente/destinazione e il protocollo di trasporto
- **Full Packet Capture**: conversazione completa tra due endpoint, non solo dati relativi alla sessione ma anche contenuto della comunicazione (crittografia permettendo)
- **Transaction Data**: artefatti prodotti dall'interazione tra endpoint spesso registrata in log o altri sistemi di raccolta dati
- **Extracted Content**: artefatti estratti dall'analisi del traffico di rete, quindi allegati da SMTP, dati di sessione, nomi DNS...
- **Statistical Data**: dati aggregati per il monitoraggio di eventi straordinari, che esulano dalla baseline
- **Alert Data**: 

**Ruoli SOC**:

**Analista Tier 1**: monitoraggio continuo degli alert, triage degli alert, monitoraggio dello stato dei sensori e degli endpoint, raccoglie dati e informazioni per Tier 2

-**Analista Tier 2**: esegue Incident Analysis in profondità correlando dati da più sorgenti, determina se sistemi critici o dati sensibili sono stati compromessi, istruisce per il ripristino della situazione, fornisce supporto per nuove metriche per Threat Detection

**Incident responder**: gestisce l'incident, mette in atto strategie di contenimento e assicura che l'Incident Response sia seguito con attenzione

- **Specialista di Forensics**: incentrato sul raccogliere e analizzare dati per scopo investigativo senza contaminare quanto raccolto

- **Specialista Malware Reverse Engineering**: analizza nel dettaglio il comportamento dei malware per determinare TTPs e IOCs, produce signatures

- **SOC Management**: gestisce risorse come personale, budget, turnazioni, tecnologie per rispettare SLA

- **Executive**: direzione generale del SOC e assicura che gli obiettivi siano rispettati


**Worflow Management System, Security Orchestration, Automation, Response**: automazione creazione ticket incidenti e automazione risposta agli incidenti