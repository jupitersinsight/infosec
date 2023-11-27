## Link:
- [**Virtual Address Space (Memory Management)**](https://learn.microsoft.com/en-us/windows/win32/memory/virtual-address-space)
- [**Virtual Address Space (Programming Guide for 64-bit Windows)**](https://learn.microsoft.com/en-us/windows/win32/winprog64/virtual-address-space)

Ogni processo ha il proprio set di memoria virtuale privata, di cui Windows mantiene una _paging table_ in cui sono associati indirizzi virtuali e fisici.
Quando un processo occupa più memoria virtuale di quanta memoria fisica gli sia stata dedicata, l'extra è salvato su disco in un file di paging.

Per i sistemi a 32bit, la dimensione massima teorica del set virtuale è pari a 4GB.  
Gli indirizzi da `0x00000000` a `0x7FFFFFFF` sono allocati ai processi, gli indirizzi da `0x8000000` a `0xFFFFFFFF` sono allocati alla gestione del sistema operativo.

Per i sistemi a 64bit, la dimensione massima teorica del set virtuale è pari a 256TB. 

Flag per alterare l'utilizzo della memoria da parte dei processi:
`increaseUserVA` - [`Address Windows Extension`](https://learn.microsoft.com/en-us/windows/win32/memory/address-windowing-extensions)