# Architettura di Windows

L'architettura di Windows si basa su due componenti principali:
- **User Mode** (Ring3): componente che gestisce i processi per conto degli utenti del sistema. Supporta operazioni con pochi privilegi e quando necessario può richiedere, tramite chiamate API ad-hoc, l'elevazione temporanea dei privilegi e accedere alla Kernel Mode per completare l'operazione in corso.
- **Kernel Mode** (Ring0): componente che gestisce i processi fondamentali di Windows. Opera con i massimi privilegi per gestire CPU e memoria.  
***ntdll.dll*** espone il kernel alla user-mode. È invocato da ***kernel32.dll***.  
A sua volta ***ntdll.dll*** si interfaccia con ***ntoskrnl.exe*** ovvero il kernel Windows. Si tratta di un file PE che espone migliaia di funzioni per interagire con l'hardware. Queste funzioni sono dei costrutti che quando invocati preparano i dati prima di passarli a funzioni di più basso livello.  
Ad esempio ***NtCreateFile*** esegue una serie di ***mov*** prima di passare il dato/valore a ***IopCreateFile***.

<img src="https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/userandkernelmode01.png" width="100%" height="auto">

Processi creati in User Mode hanno ognuno un proprio indirizzo virtuale che è limitato in dimensione e non accessibile da altre applicazioni. Se una specifica applicazione crasha, le altre applicazioni e il sistema operativo continuano a funzionare correttamente.

Per contro, i processi creati in Kernel Mode condividono lo stesso indirizzo virtuale, quindi se un processo crasha l'intero sistema operativo crasha.  
Inoltre, dal kernel sono accessibili le pagine di memoria dello user-space in quanto Windows non pone limiti di sicurezza sulle operazioni che originano dal kernel-space che hanno completo accesso al *system-space*.

`SYSENTER`, `SYSCALL` o `INT 0x2E` sono indicatori che un programma sta richiedendo accesso alla kernel-mode (accesso che avviene solo atraverso l'uso di Windows API).

Il processo **Session Manager** (Smss.exe) si occupa di avviare User e Kernel Mode all'avvio del sistema (`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\SubSystems`).  

Processo => Singolo Thread (unità più piccola riconosciuta dal SO a cui è possibile associare "tempo processore") => Da primo Thread >> Molteplici Thread che condividono spazio virtuale e contesto del processo.

Un'applicazione non può accedere direttamete a un file (oggetto) ma lo fa attraverso un handle.

## Kernel Driver Object

I *device drivers*, generalmente riferiti come *drivers*, sono pezzi di codice che permettono l'esecuzione di codice nel kernel di Windows.  
L'individuazione e l'analisi dei driver malevoli non è semplice in quanto si caricano in memoria e nessun dispositivo/applicazione interagisce direttamente con i driver. L'interazione avviene tramite *device objects*, ovvero middleman che gestiscono le richieste delle applicazioni verso i driver di riferimento.  

Un esempio: quando si inserisce una chiavetta USB nel computer, Windows si occupa di creare un *device object X:drive*. Le applicazioni che necessitano di eseguire operazioni sulla chiavetta USB interagiscono con questo *device object*. Allo stesso modo, nel momento in cui si inserisce una seconda chiavetta USB, lo stesso driver potrebbe essere coinvolto ma viene creato un secondo oggetto *Y:drive*.

Per consentire il corretto funzionamento di questo sistema, i driver devono essere caricati nel kernel così come le DLL sono caricate nei processi.

Più in dettaglio, quando un driver è caricato, la sua registrazione nel sistema avviene nella routine *DriverEntry*. Windows crea una *driver object structure* che è passata alla routine *DriverEntry*. Questa routine si occupa di riempire la *driver object structure* con le sue callback functions.  
La routine crea infine un *device* accessibile dallo user-space per l'interazione col driver.

Il kernel di Windows supporta stringhe unicode con sintassi diverse dal sistema usato in user-space: `\ROOTObject\Drive:\File`.

## Windows API

Il sistema di nomeclatura delle API di Windows è chiamato *Hungarian Notation* e consiste nel apporre un prefisso ai nomi delle funzioni/variabili che ne indichi chiaramente la natura.  
Ad esempio, se la funzione VirtualAllocEx accetta come suo parametro *n* **dwSize** si sa che si tratta di una variabile tipo DWORD.

|Prefisso e tipo|Descrizione|
|-|-|
|WORD (w)|Valore a 16bit unsigned|
|DWORD (dw)|Valore a 32bit unsigned|
|Handles (H)|Riferimento a un oggetto, esempi: HMOdule, HInstance, HKey|
|Long Pointer (LP)|Un pointer a un altro tipo, esempi: LPByte (byte), LPCSTR (carattere)|
|Callback|Rappresentano una funzione che sarà chiamata dalla API di Windows|

Oltre le Windows API, esiste un set di API non pienamente documentate da Microsoft che prendono il nome di [**Native API**](https://learn.microsoft.com/en-us/sysinternals/resources/inside-native-applications).  
Queste API funzionano a livello di kernel e permettono un accesso totale al sistema e fanno quindi gola agli sviluppatori di malware.
La DLL *ntdll.dll* è una DLL speciale che gestisce l'interazione tra spazio User e Kernel (le cui funzioni sono in *ntoskrnl.exe*).


### Handles

Gli Handle sono oggetti aperti o creati dal sistema operativo come finestre, processi, moduli, menu, file e così via.  
Gli Handle sono simili ai Pointers, in quanto fanno riferiento ad oggetti o indirizzi di memoria, ma non possono, ad esempio, essere usati in operazioni aritmetiche.  

### Funzioni/API d'interesse per il FileSystem

|Nome|Descrizione|
|-|-|
|CreateFile|Crea o apre file|
|ReadFile e WriteFile|Scrive su file o legge *n* byte del file per volta|
|CreateFileMapping e MapViewOfFile|Il primo è utile per gli hacker in quanto carica in memoria un dato file, mentre il secondo restituisce un pointer all'indirizzo di memoria dell'oggetto specificato|
|

### Funzioni/API d'interesse per il Networking
***Prima di poter usare qualsiasi funzione legata al networking è necessario chiamare la funzione *WSAStartup* per allocare le risorse necessarie (durante analisi statica o dinamica è utile impostare un breakpoint per poi analizzare le operazioni di rete).***

*da ws2_32.dll* / *Winsock API*
|Nome|Descrizione|
|-|-|
|socket|Crea un socket|
|bind|Attacca un socket a una specifica porta|
|listen|Il socket rimane in ascolto per connessioni in ingresso|
|accept|Apre una connessione a un socket remoto e accetta la connessione|
|connect| apre una connessione a un socket remoto; il socket remoto deve essere in ascolto e in attesa della connessione|
|recv|Riceve dati da un socket remoto|
|send|Invia dati a un socket remoto|

Altra DLL importante è *Wininet.dll** e la relativa API *WinINet API* per operazioni a un livello più alto (HTTP,FTP...).

## Threads e Mutexes

I processi non sono altro che *contenitori* per i thread che sono ciò che la CPU effettivamente esegue. Ogni thread è infatti una sequenza di istruzioni ben definita col proprio stack e registri, anche se condivide lo spazio di memoria con altri thread. 
Ogni thread mantiene il controllo esclusivo sulla CPU o sul core assegnato. Per permettere l'esecuzione "contemporanea" di più thread, la CPU salva in un *thread context* tutti i valori usati da un dato thread.  
A questo punto la CPU carica il +thread context* relativo a un altro thread per continuarne l'esecuzione. 

I **Mutexes** altro non sono che oggetti globali e sono spesso usati per il controllo dell'accesso a risorse condivise. Un Mutex può essere "bloccato" e usato da un solo thread per volta. Essendo i Mutexes spesso *hard-coded* sono spesso ottimi candidati per la generazione di signatures host-based.

Un thread usa la funzione `WaitForSingleObject` per prenotare l'accesso a un Mutex e la funzione `ReleaseMutex` per rilasciarlo.

## Servizi

Windows offre, come negli altri casi, funzioni dedicate per la gestione dei servizi attraverso le sue API come `OpenSCManager`, `CreateService` e `StartService`.  
In Windows esistono diversi *tipi* di servizi: con processo condiviso `SERVICE_WIN32_SHARE_PROCESS`, ovvero il codice del servizio è racchiuso in una DLL e il processo di esecuzione è condiviso con altri servizi, con processo di proprietà `SERVICE_WIN32_OWN_PROCESS`, ovvero il codice è racchiuso in un file eseguibile ed eseguito in un processo indipendente.  
Altro tipo di servizio da tenere in considerazione è `SERVICE_KERNEL_DRIVER`, ovvero il servizio è un driver.  
`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services` percorso del Registro di sistema dove sono memorizzate le informazioni passate a `CreateService` le quali sono salvate in una nuova chiave.

## COM (Component Object Model)

Interfaccia standard di Windows che permette a componenti software di richiamare codice di altri componenti software senza conoscerne le specifiche tecniche. Il framework funziona in client/server dove i client sono i componenti software mentre il server gli oggetti COM.

Gli oggetti COM sono richiamati dai programmi usando il loro GUID chiamato CLSID (class identifier) e IID (interface identifier).  
Esempio di funzionamento: un malware usa la funzione `Navigate` (che permette l'apertura del browser di sistema e la navigazione a un sito web, in questo esempio IE) parte dell'interfaccia `IWebBrowser2` implementata da Internet Explorer.

## File speciali

**File condivisi**  
Accessibli tramite `\\nomeServer\risorsa` o `\\?\nomeServer\risorsa`, dove `\\?\` indica che Windows non deve eseguire il parsing della stringa e attivare il supporto per l'accesso a file con nome lungo.  

### File accessibili tramite Namespaces

Esempio `\\.\PhysicalDisk1` permette di accedere direttamente al disco fisico 1 indipendente dal filesyestem in uso e di leggerne o modificarne i dati.  

### ADS o Alternate Data Stream

Funzionalità che permette di memorizzare in un file un secondo stream di dati che rimane nascosto fino a quando invocato o individuato e manualmente esposto.

## Filesystem NTFS

NTFS è acronimo di New Tchnology File System e di interesse ai fini di sicurezza informatica è il supporto al **Alternate Data Stream**.  
```cmd
echo "Testo nascosto">file_innocuo.txt:file_ads.txt
```

## Execution Policy

Comandi: `Get-ExecutionPolicy`, `Set-ExecutionPolicy`

**Restricted**: profilo predefinito per l'esecuzione di script che blocca l'esecuzione di script di Powershell nel sistema.

**AllSigned**: sono eseguiti solo gli script di Powershell firmati digitalmente.

**RemoteSigned**: gli script creati localmente possono essere eseguiti senza restrizioni mentre quelli scaricati da fonti terze devono essere firmati digitalmente.  
Windows considera gli script di Powershell come provenienti da fonti esterne quando il contiene ADS (Alternate Data Stream) che indica la provenienza dalla "zona Internet".

**Unrestricted**: non ci sono restrizioni all'esecuzione di script di Powershell.

## Registro di Sistema di Windows

- **HKEY_CURRENT_USER (HKCU)**: contiene informazioni associate all'utente loggato.
    - `HKCU\SOFTWARE\Classes\CLSID`: elenco di oggetti COM

- **HKEY_USERS (HKU)**: contiene informazioni su tutti gli account utente del sistema.

- **HKEY_CLASSES_ROOT (HKCR)**: contiene informazioni su associazioni di file e registrazioni OLE, Object Linking and Embedding.

- **HKEY_LOCAL_MACHINE (HKLM)**: contiene informazioni di sistema.  
    Rilevanti ai fini della gestione dei Driver e Servizi
    - `HKLM\SYSTEM\CurrentControlSet\Control`  
    - `HKLM\SYSTEM\CurrentControlSet\Enum`  
    - `HKLM\SYSTEM\CurrentControlSet\HardwareProfiles`  
    - `HKLM\SYSTEM\CurrentControlSet\Services`: elenco di servizi

    - `HKLM\SOFTWARE\Classes\CLSID`: elenco di oggetti COM

- **HKEY_CURRENT_CONFIG (HKCC)**: contiene informazioni sul profilo hardware in uso.

