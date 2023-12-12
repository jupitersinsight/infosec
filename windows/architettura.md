## Architettura di Windows

L'architettura di Windows si basa su due componenti principali:
- **User Mode**: componente che gestisce i processi per conto degli utenti del sistema. Supporta operazioni con pochi privilegi e quando necessario può richiedere, tramite chiamate API ad-hoc, l'elevazione temporanea dei privilegi e accedere alla Kernel Mode per completare l'operazione in corso.
- **Kernel Mode**: componente che gestisce i processi fondamentali di Windows. Opera con i massimi privilegi per gestire CPU e memoria.

<img src="https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/userandkernelmode01.png" width="100%" height="auto">

Processi creati in User Mode hanno ognuno un proprio indirizzo virtuale che è limitato in dimensione e non accessibile da altre applicazioni. Se una specifica applicazione crasha, le altre applicazioni e il sistema operativo continuano a funzionare correttamente.

Per contro, i processi creati in Kernel Mode condividono lo stesso indirizzo virtuale, quindi se un processo crasha l'intero sistema operativo crasha.

Il processo **Session Manager** (Smss.exe) si occupa di avviare User e Kernel Mode all'avvio del sistema (`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\SubSystems`).  

Processo => Singolo Thread (unità più piccola riconosciuta dal SO a cui è possibile associare "tempo processore") => Da primo Thread >> Molteplici Thread che condividono spazio virtuale e contesto del processo.

Un'applicazione non può accedere direttamete a un file (oggetto) ma lo fa attraverso un handle.

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

## Windows Registry

- **HKEY_CURRENT_USER (HKCU)**: contiene informazioni associate all'utente loggato.

- **HKEY_USERS (HKU)**: contiene informazioni su tutti gli account utente del sistema.

- **HKEY_CLASSES_ROOT (HKCR)**: contiene informazioni su associazioni di file e registrazioni OLE, Object Linking and Embedding.

- **HKEY_LOCAL_MACHINE (HKLM)**: contiene informazioni di sistema.

- **HKEY_CURRENT_CONFIG (HKCC)**: contiene informazioni sul profilo hardware in uso.
