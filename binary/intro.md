**Assembler** : traduttore di linguaggio macchina, programma che traduce assembly in linguaggio macchina.  
**Compiler**: converte un linguaggio di alto-livello in linguaggio macchina occupandosi di adattare il risultato delle conversione all'architettura del processore.  

**objdump** : tool per analizzare il file eseguibile derivato dalla compilazione di _codice.c_, ovvero il codice macchina.

Utile avere il _codice\_sorgente.c_ ma è ancora più utile saper leggere il contenuto del file eseguibile, in quanto è **questo** che viene **eseguito**.

Codice C usato per gli esempi:

```c
#include <stdio.h>

int main()
{
   int i;
   for(i=0;i<10;i++)
      printf("Hello, World!\n");
   return 0;
}

```

```assembly
----------objdump -D firstprog | grep -A20 main.:
08048374 <main>:
 8048374:       55                      push   %ebp
 8048375:       89 e5                   mov    %esp,%ebp
 8048377:       83 ec 08                sub    $0x8,%esp
 804837a:       83 e4 f0                and    $0xfffffff0,%esp
 804837d:       b8 00 00 00 00          mov    $0x0,%eax
 8048382:       29 c4                   sub    %eax,%esp
 8048384:       c7 45 fc 00 00 00 00    movl   $0x0,0xfffffffc(%ebp)
 804838b:       83 7d fc 09             cmpl   $0x9,0xfffffffc(%ebp)
 804838f:       7e 02                   jle    8048393 <main+0x1f>
 8048391:       eb 13                   jmp    80483a6 <main+0x32>
 8048393:       c7 04 24 84 84 04 08    movl   $0x8048484,(%esp)
 804839a:       e8 01 ff ff ff          call   80482a0 <printf@plt>
 804839f:       8d 45 fc                lea    0xfffffffc(%ebp),%eax
 80483a2:       ff 00                   incl   (%eax)
 80483a4:       eb e5                   jmp    804838b <main+0x17>
 80483a6:       b8 00 00 00 00          mov    $0x0,%eax
 80483ab:       c9                      leave  
 80483ac:       c3                      ret    
 80483ad:       90                      nop    
 80483ae:       90                      nop
```

I numeri esadecimali a sinistra sono gli indirizzi di memoria (come in una strada con case su entrambi i lati).  
La memoria è una collezione di byte di archiviazione temporanea numerati con indirizzi.  

Il linguaggio **assembly**, quindi le istruzioni sulla destra, non è altro che un modo per i programmatori per rappresentare le istruzioni in linguaggio macchina passate al processore.  

L'estratto di output poco sopra usa la sinstassi AT&T mentre quello che segue usa la sintassi Intel.  

```assembly
----------objdump -M intel -D firstprog | grep -A20 main.:
08048374 <main>:
 8048374:       55                      push   ebp
 8048375:       89 e5                   mov    ebp,esp
 8048377:       83 ec 08                sub    esp,0x8
 804837a:       83 e4 f0                and    esp,0xfffffff0
 804837d:       b8 00 00 00 00          mov    eax,0x0
 8048382:       29 c4                   sub    esp,eax
 8048384:       c7 45 fc 00 00 00 00    mov    DWORD PTR [ebp-4],0x0
 804838b:       83 7d fc 09             cmp    DWORD PTR [ebp-4],0x9
 804838f:       7e 02                   jle    8048393 <main+0x1f>
 8048391:       eb 13                   jmp    80483a6 <main+0x32>
 8048393:       c7 04 24 84 84 04 08    mov    DWORD PTR [esp],0x8048484
 804839a:       e8 01 ff ff ff          call   80482a0 <printf@plt>
 804839f:       8d 45 fc                lea    eax,[ebp-4]
 80483a2:       ff 00                   inc    DWORD PTR [eax]
 80483a4:       eb e5                   jmp    804838b <main+0x17>
 80483a6:       b8 00 00 00 00          mov    eax,0x0
 80483ab:       c9                      leave  
 80483ac:       c3                      ret    
 80483ad:       90                      nop    
 80483ae:       90                      nop  
```

Ogni famiglia/gruppo di processori ha il suo set di **registri**, ovvero set di variabili speciali. La maggior parte delle istruzioni usa questi registri per leggere e scrivere dati.  

I programmi chiamati **debugger** servono lo scopo di analizzare i vari step di funzionamento di un programma compilato, di esaminare la memoria di un programma e di visualizzare i registri del processore.  
GCC include un debugger chiamato GDB.  

```assembly
----------gdb -q ./firstprog 
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) break main
Breakpoint 1 at 0x804837a
(gdb) run
Starting program: /home/jito/firstprog 

Breakpoint 1, 0x0804837a in main ()
(gdb) info registers
eax            0xbffff8e4       -1073743644
ecx            0x48e0fe81       1222704769
edx            0x1      1
ebx            0xb7fd6ff4       -1208127500
esp            0xbffff850       0xbffff850
ebp            0xbffff858       0xbffff858
esi            0xb8000ce0       -1207956256
edi            0x0      0
eip            0x804837a        0x804837a <main+6>
eflags         0x200286 [ PF SF IF ID ]
cs             0x73     115
ss             0x7b     123
ds             0x7b     123
es             0x7b     123
fs             0x0      0
gs             0x33     51
````

**break main** : inserisce breakpoint su `main()`, quindi il codice una volta avviato il programma col comando `run` si ferma al breakpoint, prima dell'esecuzione della funzione principale.  
Sulla sinistra ci sono i registri del processore di cui i primi 4 (EAX, ECX, EDX, EBX) sono considerati per uso generico ma principalmente come variabili temporanee.  
- **EAX** : Accumulator
- **ECX** : Counter
- **EDX** : Data
- **EBX** : Base

I registri ESP, EBP, ESI, EDI sono anche questi considerati come registri per uso generico ma sono definiti _pointers_ e _indexes_ perché:
- **ESP** : Stack Pointer
- **EBP** : Base Pointer
- **ESI** : Source Index
- **EDI** : Destination Index

I primi due registri sono chiamati _pointers_ perché immagazzinano informazioni sulle posizioni 32-bit, puntanto quindi a specifiche aree di memoria. Gli altri due registri sono comunemente usati per puntare a sorgente e destinazione di dati quando c'è necessità di lettura o scrittura.  

Il registro EIP (**Instruction Pointer**) punta all'istruzione in quel momento in lettura da parte del processore. 

Il registro EFLAGS contiene diversi flag che sono usati per comparazione e segmentazione della memoria.

Per cambiare la sintassi in GDB `set disassemlby intel` o `set dis intel` (persistente: `echo "set dis intel" > ./.gdbinit`).  

L'istruzione assembly secondo la sinstassi Intel è `operazione <destinazione>, <sorgente>. 

Es. 1  
```assembly
8048375:       89 e5                   mov    ebp,esp
8048377:       83 ec 08                sub    esp,0x8
```
Sposta (mov) il valore da ESP a EBP. Sottrae (sub) 8 da ESP.

Es. 2
```assembly
804838b:       83 7d fc 09             cmp    DWORD PTR [ebp-4],0x9
804838f:       7e 02                   jle    8048393 <main+0x1f>
8048391:       eb 13                   jmp    80483a6 <main+0x32>
```
Compara due valori (cmp), ovvero 9 e EBP-4. Se la comparazione "salta se minore di o uguale a" (jle) ritorna un valore vero l'esecuzione salta all'indirizzo 0x8048393. Altrimenti si effettua il salto (jmp) all'indirizzo 0x80483a6.  
Quando si compila un programma, il compiler aggiunge istruzioni definite _function prologue_, per creare spazio necessario per l'esecuzione del codice compilato.  

```assembly
(gdb) disassemble main
Dump of assembler code for function main:
```

**[inizio function prologue]**

```assembly
0x08048374 <main+0>:    push   ebp
0x08048375 <main+1>:    mov    ebp,esp
0x08048377 <main+3>:    sub    esp,0x8
0x0804837a <main+6>:    and    esp,0xfffffff0
0x0804837d <main+9>:    mov    eax,0x0
0x08048382 <main+14>:   sub    esp,eax
```

**[fine function prologue]**

```assembly
0x08048384 <main+16>:   mov    DWORD PTR [ebp-4],0x0 ### i=0 (variabile i occupa 4 byte), è assegnato il valore 0 a ebp-4
0x0804838b <main+23>:   cmp    DWORD PTR [ebp-4],0x9 ### comparazione tra valore 9 e ebp-4
0x0804838f <main+27>:   jle    0x8048393 <main+31> ### se il valore dell'operazione precedente (memorizzato nel registro EFLAGS) è minore o uguale a 9 in questo caso, salta all'indirizzo 8048393
0x08048391 <main+29>:   jmp    0x80483a6 <main+50> ### salto non condizionato da if-else, salta all'indirizzo 80483a6
	 <main+31>:   mov    DWORD PTR [esp],0x8048484 ### "Hello World!\n" --> vedi sotto
0x0804839a <main+38>:   call   0x80482a0 <printf@pltman > ### chiama la funzione di printf
0x0804839f <main+43>:   lea    eax,[ebp-4] ### 
0x080483a2 <main+46>:   inc    DWORD PTR [eax]
0x080483a4 <main+48>:   jmp    0x804838b <main+23>
0x080483a6 <main+50>:   mov    eax,0x0
0x080483ab <main+55>:   leave  
0x080483ac <main+56>:   ret 
```

`info register <registro>`

Per esaminare/analizzare indirizzi di memoria usare i comandi `x/[codifica] 0x[indirizzo di memoria] o $[registro]`.  
Codifiche: 
- **o** : ottale
- **x** : esadecimale
- **u** : non specificato, usa base10
- **t** : binario

Anteponendo un valore numerico alla codifica è possibile esaminare più unità per indirizzo di memoria.
```assembly
(gdb) x/12o 0x8048384
0x8048384 <main+16>:    077042707       020300000000    017602376175    030704765402
0x8048394 <main+32>:    020441022004    0172004004      021577777777    077776105
0x80483a4 <main+48>:    056162753       031100000000    022044110303    013571304525
```
La dimensione predefinita di un'unità è di 4 byte e viene definita **word**.  
Postponendo una codifica per la dimensione è possibile cambiare la dimensione delle unità nell'output.  
Codifiche:
- **b** : singolo byte
- **h** : 2 byte
- **w** : 4 byte
- **g** : 8 byte

In alcuni casi il termine **word** identifica anche unità di 2 byte, mentre un valore di 4 byte è definito come **double word** o **DWORD**.  

Memorizzazione ordine byte (in base all'hardware):
**little-endian** : memorizzazione/trasmissione inizia dal byte meno significativo (estremità più piccola)
**big-endian** : memorizzazione/trasmissione inizia dal byte più significativo (estremità più grande)
Il debugger riconosce come il valore deve essere memorizzato e interpretato, mostra il byte nell'ordine corretto quando è nella sua giusta dimensione (esempio 4 byte).  
Quando invece è impostata una dimensione diversa nella visualizzazione, ad esempio 2 byte, i valori sono invertiti affinché il valore decimale risulti corretto.
```assembly
0x8048384 <main+16>:    0x45c7  0x00fc  0x0000  0x8300
(gdb) x/4xw $eip
0x8048384 <main+16>:    0x00fc45c7      0x83000000      0x7e09fc7d      0xc713eb02
```

`print` esegue calcoli a richiesta, es. `print $ebp -4`.  
Assegnazione valore a variabile `$1 = [valore]`.  
Il comando examine, x, supporta l'argomento **i**, _instructios_, che mostra in output la memoria disassemblata come linguaggio assembly.  
Es:
```assembly
(gdb) x/10i $eip
0x8048384 <main+16>:    mov    DWORD PTR [ebp-4],0x0
0x804838b <main+23>:    cmp    DWORD PTR [ebp-4],0x9
0x804838f <main+27>:    jle    0x8048393 <main+31>
0x8048391 <main+29>:    jmp    0x80483a6 <main+50>
0x8048393 <main+31>:    mov    DWORD PTR [esp],0x8048484
0x804839a <main+38>:    call   0x80482a0 <printf@plt>
0x804839f <main+43>:    lea    eax,[ebp-4]
0x80483a2 <main+46>:    inc    DWORD PTR [eax]
0x80483a4 <main+48>:    jmp    0x804838b <main+23>
0x80483a6 <main+50>:    mov    eax,0x0
0x80483ab <main+55>:    leave  
0x80483ac <main+56>:    ret 
```

```assembly
(gdb) x/2xw 0x8048484
0x8048484:      0x6c6c6548      0x57202c6f
(gdb) x/6xb 0x8048484
0x8048484:      0x48    0x65    0x6c    0x6c    0x6f    0x2c
(gdb) x/14xb 0x8048484
0x8048484:      0x48    0x65    0x6c    0x6c    0x6f    0x2c    0x20    0x57
0x804848c:      0x6f    0x72    0x6c    0x64    0x21    0x0a

```
Valore hex da tabella ASCII:
- **0x48** : H
- **0x65** : e
- **0x6c** : l
- **0x6f** : o
- **0x2c** : ,
- **0x20** : SPAZIO
- **0x57** : W
- **0x72** : r
- **0x64** : d
- **0x21** : !
- **0x0a** : LF (\n)

```assembly
(gdb) x/14cb 0x8048484
0x8048484:      72 'H'  101 'e' 108 'l' 108 'l' 111 'o' 44 ','  32 ' '  87 'W'
0x804848c:      111 'o' 114 'r' 108 'l' 100 'd' 33 '!'  10 '\n'
```
```assembly
(gdb) x/s 0x8048484
0x8048484:       "Hello, World!\n"
```

