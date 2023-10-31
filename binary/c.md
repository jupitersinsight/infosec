Lista = array o buffer = lista di _n_ elementi.
Es.
```c
int main()
{
   char str_a[20];
   str_a[0] = 'H';
   str_a[1] = 'e';
   str_a[2] = 'l';
   str_a[3] = 'l';
   str_a[4] = 'o';
   str_a[5] = ',';
   str_a[6] = ' ';
   str_a[7] = 'W';
   str_a[8] = 'o';
   str_a[9] = 'r';
   str_a[10] = 'l';
   str_a[11] = 'd';
   str_a[12] = '!';
   str_a[13] = '\n';
   str_a[14] = 0; //Null-byte per terminare la stringa
   printf(str_a);
}   
```

Versione abbreviata del codice c:
```c
int main()
{
   char str_a[20];
   strcpy(str_a, "Hello, World!\n"); //La funzione strcpy(<destinazione>, <sorgente>) itera la stringa sorgente e copia ogni byte nella destinazione. Si ferma al null-byte
   printf(str_a);
}
```

In C le variabili numeriche possono essere _signed_, _unsigned_, _short_ e _long_.  
- **unsigned** : solo numeri positivi, da 0 a 2^32 (4'294'967'296)
- **signed** : numeri positivi e negativi, da -2'147'483'648 a 2'147'483'648 (uno dei bit contiene il flag che determina se il valore è positivo o negativo).  
- **short** : dimensione ridotta per valori numerici
- **long** : dimensione estesa per valori numerici

_Two's complement_ metodo di memorizzazione dei numeri negativi:
- numero positivo in binario
- inversione dei bit
- somma dei bit invertiti + 1

= rappresentazione negativa del valore positivo originale

Funzione _sizeof()_ restituisce la dimensione del valore passato calcolato in byte.  

**Pointers** : variabili dichiarate anteponendo l'asterisco `*` al nome della variabile, puntano agli indirizzi di memoria (come un collegamento al contenuto di altre variabili).  
**EIP** = Instruction Pointer -> punta all'istruzione corrente e contiene il suo indirizzo in memoria.  
Per vedere l'effettiva informazione memorizzata in una variabile _pointer_ si può usare l'operatore _address-of_ (_unary operator_ ovvero accetta solo un argomento).  
L'operatore è il simbolo `&`, anteposto al nome della variabile.  

```assembly
(gdb) list
6          char *pointer;
7          char *pointer2;
8
9          strcpy(str_a, "Hello, World!\n");
10         pointer = str_a;
11         printf(pointer);
12
13         pointer2 = pointer + 2;
14         printf(pointer2);
15
(gdb) x/xw pointer // Analisi della variabile pointer com stringa. <Indirizzo>: <valore>.
0xbffff820:     0x6c6c6548
(gdb) x/sw pointer
0xbffff820:      "Hello, World!\n"
(gdb) x/xw &pointer //Variabile pointer risiede all'indirizzo 0xbffff81c che contiene come valore l'indirizzo 0xbffff820
0xbffff81c:     0xbffff820
(gdb) print &pointer
$1 = (char **) 0xbffff81c
(gdb) print pointer
$2 = 0xbffff820 "Hello, World!\n" //Indirizzo 0xbffff820 contiene la stringa "Hello, World!\n".  
```

L'operatore `*`, chiamato _dereference_, ritorna il contenuto dell'indirizzo invece del solo indirizzo.  

Parametri in stringhe:
- **%s**: stringa
- **%d**: numeri interi
- **%u**: decimale unsigned (solo numeri positivi. in caso di numero negativo i bit a 0 sono girati a 1 e il valore risutla essere un numero alto)
- **%x**: esadecimale
- **%f**: numeri con virgola
- **%CIFRAd/s/u/x**: specifica la dimensione minima
- **%0CIFRAd/s/u/x**: come sopra ma c'è padding di 0
- **%p**: indirizzo di memoria (scorciatoia `0x%08x`)

**Typecast** = trasformazione temporanea del tipo di variabile in altro tipo, es. int x >> (float) x ... la trasformazione vale solo per l'operazione specifica.

**void pointer** = variabile di tipo non specificato per contenere indirizzi di memoria.  
Per ogni suo utilizzo, ad esempio in for loop, bisogna usare typecasting per dichiarare per quell'operazione che tipo di valore assume.

**static variable** = variabili contestualizzate all'interno di un contesto di una specifica funzione.  
Variabili statiche con lo stesso nome possono esistere, ad esempio all'interno di nested functions, ma fanno riferimento a indirizzi di memoria diversi.  
Sono inizializzate una volta sola.    


Segmentazione della memoria = la memoria di un programma compilato è diviso in 5 segmenti: text, data, bss, heap e stack.  
Il _text segment_, o _code segment_ include le istruzioni in linguaggio assembly ed è di sola lettura, ha una dimensione fissa (non sono previste modifiche al codice). . La sola lettura permette l'esecuzione di più istanze e impedisce che il codice sia modificato e impedisce tentativi di manomissione del codice (qualsiasi tentativo comporta la chiusura del programma).  

I segmenti _data_ e _bss_ contengono rispettivamente le variabili global e static inizializzate e quelle non inizializzate.  

Il segmento _heap_ è un segmento di memoria direttamente controllabile dal programmatore. Questo segmento, a differenza di text, non ha una dimensione fissa. L'heap cresce verso il basso, verso indirizzi più alti.  

Il segmento _stack_ ha una dimensione variabile il cui scopo è di memorizzare variabili locali (GDB backtrace analizza questo segmento). Il segmento stack contiene diversi _stack frame_, ognuno dei quali contiene informazioni per il ritorno dell'EIP , tutte le variabili locali e ricorda tutte le variabili passate.  
Le informazioni nello stack seguono l'ordinamento _First In Last Out (FILO)_. Push  = inserisci, Pop = rimuovi.  
Il registro ESP tiene traccia dell'ultimo indirizzo nello stack.  
A differenza dell'heap, lo stack cresce verso l'alto verso gli indirizzi più bassi. 
EBP (frame pointer (FP) o local base (LB) pointer) = usato per fare riferimento a variabili locali nello stack frame.

{Composizione StackFrame}: 
- Parametri di una funzione e le sue variabili locali
- SFP = Saved Frame Pointer riporta EBP  al valore precedente  
- Return address = porta EIP al valore dell'istruzione successiva dopo la chiamata di funzione  

<img src="https://textbook.cs161.org/assets/images/memory-safety/x86/memory-sections.png" width="100%" height="auto" alt="Stack Frame"/>  

Esempio di segmentazione della memoria usando C:

```c
#include <stdio.h>

int global_var; // variabile globale alla funzione, non inizializzata = bss
int global_initialized_var = 5; // variabile globale alla funzione inizializzata = data

void function() {
   int stack_var; // variabile locale alla funzione, non inizializzata = stack
   printf("La variabile 'stack_var' locale alla funzione 'function()' si trova all'indirizzo %p\n", &stack_var);
}

int main(){
   int stack_var; // variabile locale alla funzione, non inizializzata = stack
   static int static_initialized_var = 5; // variabile statica inizializzata = data
   static int static_var; // variabile statica non inizializzata = bss
   int *heap_var_ptr; // variabile/pointer = heap

   heap_var_ptr = (int *) malloc(4); // funzione malloc() alloca memoria ma il valore restituito è di tipo void e serve typecast per convertire la variabile in altro tipo, in questo caso int.   
   
   // DATA
   printf("La variabile 'global_initialized_var' si trova all'indirizzo %p\n", &global_initialized_var);
   printf("La variabile 'static_initialized_var' si trova all'indirizzo %p\n\n", &static_initialized_var);

   // BSS
   printf("La variabile 'global_var' si trova all'indirizzo %p\n", &global_var);
   printf("La variabile 'static_var' si trova all'indirizzo %p\n\n", &static_var);

   // HEAP
   printf("La variabile 'heap_var_ptr' si trova all'indirizzo %p\n\n", &heap_var_ptr);

   // STACK
   printf("La variabile 'stack_var' locale alla funzione 'main()' si trova all'indirizzo %p\n", &stack_var);
   function();
}
```

**malloc(dimensione in byte)** = riserva la quantità specificata nel segmento heap e ritorna un void pointer.  
**free(indirizzo di memoria heap)** = libera la memoria occupata all'indirizzo specificato.

L'apertura di file in c può avvenire tramite _file descriptors_ (funzioni I/O low level) o _file streams_(funzionio I/O high level costruite su low level).  
Prendendo in esame il primo (file descriptor), un file descriptor è un numero intero che identifica in maniera univoca un file tra i file aperti (pointer al file aperto).  
Operazioni:
- **open**(pointer_a_filename, flags_che_regolano_accesso_al_file) = return file_descriptor (fd)
- **read**(fd, pointer_a_dati_da_leggere, numero_byte_da_leggere)
- **write**(fd, pointer_a_dati_da_scrivere, numero_byte_da_scrivere)
- **close**(fd)

Esempio:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>	// Assieme a sys/stat.h serve per poi richiamare i flag necessari al funzionamento di open()
#include <sys/stat.h>	//

void usage(char *prog_name, char *filename){
   printf("Utilizzo: %s <testo da aggiungere  %s>\n", prog_name, filename);
   exit(0);
}

void fatal(char *);
void *ec_malloc(unsigned int);

int main(int argc, char *argv[]){
   int fd;
   char *buffer, *datafile;

   buffer = (char *) ec_malloc(100);
   datafile = (char *) ec_malloc(20);
   strcpy(datafile, "/tmp/notes");

   if(argc < 2)
      usage(argv[0], datafile);
   
   strcpy(buffer, argv[1]);
   printf("[DEBUG] buffer @ %p : '%s'\n", buffer, buffer);
   printf("[DEBUG] datafile # %p : '%s'\n", datafile, datafile);

   strncat(buffer, "\n", 1);

   fd = open(datafile, O_WRONLY|O_CREAT|O_APPEND, S_IRUSR|S_IWUSR);  //flag usati con bitwise operator OR in quanto sono modificati singoli bit

   if(fd == -1)
      fatal("in main() aprendo un file");
   printf("[DEBUG] file descriptor is %d\n", fd);

   if(write(fd, buffer, strlen(buffer)) == -1)
      fatal("in main() scrivendo il contenuto del buffer nel file");

   if(close(fd) == -1)
      fatal("in main() chiudendo il file");

   printf("Note salvate\n");
   free(buffer);
   free(datafile);
}

void fatal(char *message){
   char error_message[100];

   strcpy(error_message, "[!!] Errore critico ");
   strncat(error_message, message, 83);
   perror(error_message);
   exit(-1);
}

void *ec_malloc(unsigned int size){
   void *ptr;
      ptr = malloc(size);
   if(ptr == NULL)
      fatal("in ec_malloc() allocando memoria");
   return ptr;
}
```

**fcntl.h**

- O_RDONLY : apre in sola lettura  
- O_WRONLY : apre in sola scrittura  
- O_RDWR : apre in lettura e scrittura  
- O_APPEND : scrive alla fine del file  
- O_TRUNC : se il file esiste, lo tronca a lunghezza 0  
- O_CREAT : crea il file se non esiste  

**sys/stat.h**

- S_IRUSR : lettura per utente (owner)  
- S_IWUSR : scrittura per utente (owner)  
- S_IXUSR : esecuzione per utente (owner)  
- S_IRGRP : lettura per gruppo  
- S_IWGRP : scrittura per gruppo  
- S_IXGRP : esecuzione per gruppo  
- S_IROTH : lettura per altri  
- S_IWOTH : scrittura per altri  
- S_IxOTH : esecuzione per altri  

**#include <risorsa.h>** : cerca il file nella cartella di default, /usr/include  
**#include "risorsa.h"** : cerca il file nella cartella dove risiede il programma  

**Structs**: variabili che contengono altre variabili. Una volta definite in file in _/usr/include_, diventano tipi utilizzabili di variabili.

I pointer si possono usare anche per richiamare funzioni, es:
```c
#include <stdio.h>

int func_one(){
   printf("This is function one\n");
   return 1;
}

int func_two(){
   printf("This is function two\n");
   return 2;
}

int main(){
   int value;
   int (*function_ptr) (); // Pointer con typecast integer

   function_ptr = func_one; // Pointer punta all'indirizzo della funzione
   printf("function_ptr is %p\n", function_ptr);
   value = function_ptr();
   printf("value is %d\n", value);

   function_ptr = func_two;
   printf("function_ptr is %p\n", function_ptr);
   value = function_ptr();
   printf("value is %d\n", value);
}
```


