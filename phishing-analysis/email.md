# Header

Il campo **Received** non può essere modificato/spoofed e riporta i server da cui è passata l'e-mail. Le entries sono riportate dal basso verso l'alto, ovvero dal primo server (basso) attraversato all'ultimo (alto).

Verifica vdell'indirizzo IP di origine può rivelare informazioni utili sulla geolocalizzazione del mittente, specialmente quando il dominio del mittente è localizzato in una specifica regione geografica (ad esempio, una e-mail della Agenzia delle Entrate inviata dalla Turchia).  

È utile poi verificare, usando strumenti utili come "SPF Record Checker", se per il dominio (@something.com) esistono record SPF validi e quindi determinare se l'e-mail è spoofed o no.  
I record SPF possono essere raggirati nel momento in cui il Threat Actor entra in possesso di e-mail esfiltrate in un altro attacco e usa gli header "autentici" per crearne di nuovi e dare legittimità alla conversazione agli occhi dei controlli automatici, vedi SPF record.

Questa tecnica di **Conversation Hijacking** si ritrova anche in e-mail che presentano frammenti di conversazioni passate, sempre frutto di attacchi precedenti o legati a terze parti ma con cui si ha avuto uno scambio di e-mail, usate per rendere il phishing più difficile da riconoscere.


# URL

Spesso le URL hanno una vita corta in quanto servono per attacchi spot legati a microcampagne di phishing.  
Tool online come **Phishtank** sono utili per rinoscere URL di phishing già note.


# Allegati

