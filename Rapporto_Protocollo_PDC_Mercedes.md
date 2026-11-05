PDC Mercedes
Mappa del protocollo seriale

PIC16F917 · Firmware PIC16F917_benz2.hex | Revisione 1 · 5 novembre 2026

Rapporto ricavato dall’HEX e dallo schema forniti. Questa scheda impiega un **protocollo seriale proprietario tramite transceiver LIN**. Gli identificatori descritti sotto sono header seriali, non ID CAN. Il percorso ricostruito non usa il framing LIN standard break / 0x55 / protected identifier.

| Parametro | Valore / significato |

| --- | --- |

| Interfaccia | MCP2021A identificato nello schema; RX su RC7, TX su RC6; RB4 partecipa all’abilitazione. |

| Velocità | Circa 4800 baud: SPBRG=63, BRGH=1; baud = Fosc / 1024. Quarzo confermato circa 4,91 MHz; con 4,9152 MHz il valore è esattamente 4800. |

| Dati / parità | 8 bit dati più parità pari, realizzata con il nono bit UART. Tutti i byte del pacchetto hanno parità, incluso il checksum. |

| Stop bit | 4800 8E1 è un’impostazione candidata coerente con il ricevitore. Il numero di stop bit emesso dal trasmettitore esterno non è misurabile dal solo HEX. |

| Modalità corrente | EEPROM indice 5 = 2: la descrizione applicativa principale di questo rapporto riguarda la modalità 2. |




## Struttura del pacchetto


**H · D0 … D(N−1) · S**
H = header di un byte; N = numero di byte dati, da 1 a 4; S = checksum. La lunghezza totale è da 3 a 6 byte. D0 indica sempre il primo byte dati, non l’header.

| Bit di H | Campo / formula |

| --- | --- |

| 7…6 | Numero di byte dati: N = (H >> 6) + 1. Valori 00/01/10/11 → 1/2/3/4 byte dati. |

| 5…4 | Gruppo = (H >> 4) & 3. |

| 3…0 | Tipo = H & 0x0F. La sua accettazione dipende dalla modalità EEPROM. |



**S = (H + D0 + … + D(N−1)) &amp; 0xFF**
È una somma modulo 256: il checksum non è complementato e non è un CRC. I bit di parità non entrano nella somma.


## Gruppi gestiti dal dispatcher — modalità 2


| Gruppo / header | Dati e destinazione osservata |

| --- | --- |

| 0: C1, C2, C3, CD, CE, CF | 4 byte dati. RAM 0x12D…0x130; percorso LED e richiesta sonora descritto a pagina 2. |

| 1: D1, D2, D3 | 4 byte dati. RAM 0x139…0x13C. Significato fisico dei campi non ancora completato. |

| 2: 21, 22, 23 | 1 byte dati. RAM 0x128. Significato fisico non ancora completato. |

| 3: 31 / 71 / F1 | Rispettivamente 1 / 2 / 4 byte: RAM 0x129 / 0x12B…0x12C / 0x135…0x138. Il tipo deve essere 1. |



Gli header sono esadecimali. Un pacchetto con checksum valido non è necessariamente un comando accettato dall’applicazione: per esempio B1 codifica 3 byte dati, ma tale combinazione non è gestita nel gruppo 3. Dopo un intervallo di almeno sei tick, rilevato al byte successivo, il ricevitore può azzerare il frame parziale; il tempo nominale è circa 6 ms, non misurato sulla scheda.


---



## Gruppo 0 — LED e richiesta sonora in modalità 2


Questo è il percorso applicativo ricostruito più in dettaglio. Richiede sempre **quattro byte dati**, anche quando alcuni campi non vengono utilizzati: H D0 D1 D2 D3 S.

| Header | Aggiornamento dei LED |

| --- | --- |

| C1 | Aggiorna entrambi i banchi A e B. |

| C2 | Aggiorna soltanto il banco A. |

| C3 | Aggiorna soltanto il banco B. |

| CD / CE / CF | Nel percorso di gruppo 0 in modalità 2 aggiornano entrambi i banchi. In altre modalità esistono eccezioni: non vanno generalizzati a tutto il firmware. |



| Campo | Interpretazione ricostruita |

| --- | --- |

| D0, bit 7…4 | Indice del livello LED del banco A, da 0 a 15. Non è un valore direttamente espresso in centimetri. |

| D0, bit 3…0 | Indice del livello LED del banco B, da 0 a 15. |

| D1 | Non utilizzato dal percorso di gruppo 0 in modalità 2. È impiegato per i LED in altre modalità. |

| D2, due nibble | Indice sonoro = max(D2 & 0x0F, D2 >> 4). I due nibble alimentano un’unica richiesta sonora. |

| D3 | Non utilizzato dal percorso di gruppo 0 in modalità 2. È impiegato per il suono in modalità 4. |



La selezione dell’header C2/C3 limita il banco LED aggiornato, ma il calcolo dell’indice sonoro viene comunque eseguito sul D2 del pacchetto accettato. Non si può quindi usare C2/C3 assumendo che il campo sonoro sia ignorato.


## Conversione del livello in pattern LED


| Livello (dec.) | Pattern (hex) | Bit attivi nel pattern |

| --- | --- | --- |

| 0 / 1 | 00 | Nessuno. |

| 2 | 01 | Bit 0. |

| 3 | 03 | Bit 0…1. |

| 4 | 07 | Bit 0…2. |

| 5 / 6 / 7 / 8 | 0F / 1F / 3F / 7F | Rispettivamente bit 0…3 / 0…4 / 0…5 / 0…6. |

| 9 | FF | Tutti gli otto bit. |

| 10 | C0 | Soltanto bit 6 e 7. |

| 11…15 | 00 | Nessuno, secondo la tabella del renderer. |



Un bit del pattern a 1 richiede l’accensione del LED corrispondente. Le uscite fisiche sono **attive basse**: per accendere il LED il codice porta il pin a 0. La posizione fisica precisa va confrontata con il cablaggio; A e B sono nomi assegnati durante l’analisi.


## Luminosità e limiti dell’interpretazione


Il firmware applica ai LED una regolazione software a dieci fasi, con impostazioni e limiti della configurazione. Il gruppo 0 descritto qui non fornisce un campo di luminosità identificato come nel messaggio CAN 0x635 del progetto Skoda. Non sono ricostruite le distanze reali associate agli indici né la periodicità dei messaggi della centralina trasmittente.


---



## Corrispondenza dei bit con i pin LED


| Bit del pattern | Banco A | Banco B |

| --- | --- | --- |

| 0 | RD4 | RD3 |

| 1 | RB3 | RC0 |

| 2 | RB2 | RC1 |

| 3 | RB1 | RC2 |

| 4 | RB0 | RC3 |

| 5 | RD7 | RD0 |

| 6 | RD6 | RD1 |

| 7 | RD5 | RD2 |




## Tabella dei conteggi sonori


Indice ricavato dal massimo dei due nibble di D2. La tabella contiene conteggi interni: **non sono frequenze acustiche in Hz né durate misurate in millisecondi**. RC5 è collegato al transistor del cicalino; RE1 al circuito di misura.

| Indice | Conteggio | Indice | Conteggio |

| --- | --- | --- | --- |

| 0 | 0 | 8 | 216 |

| 1 | 976 | 9 | 194 |

| 2 | 650 | 10 | 96 |

| 3 | 488 | 11 | 64 |

| 4 | 390 | 12 | 48 |

| 5 | 325 | 13 | 38 |

| 6 | 278 | 14 | 32 |

| 7 | 243 | 15 | 1100 |



L’indice 15 imposta anche un flag speciale. La generazione completa del suono, l’inviluppo e i feedback analogici non sono ancora interamente ricostruiti. L’indice 0 porta il conteggio tabellare a zero; la sequenza fisica di arresto del cicalino non è stata verificata.


## Esempi sintetici — byte esadecimali


| Pacchetto completo | Effetto richiesto in modalità 2 |

| --- | --- |

| C1 34 00 55 00 4A | A: livello 3 → pattern 03; B: livello 4 → pattern 07; indice sonoro 5. |

| C2 34 00 55 00 4B | Solo banco A: livello 3; indice sonoro 5. Il banco B non viene aggiornato. |

| C3 34 00 55 00 4C | Solo banco B: livello 4; indice sonoro 5. Il banco A non viene aggiornato. |

| C1 99 00 00 00 5A | Pattern FF per entrambi i banchi; indice sonoro 0. |

| C1 AA 00 00 00 6B | Pattern C0 per entrambi i banchi; indice sonoro 0. |

| C1 00 00 00 00 C1 | Pattern 00 per entrambi i banchi; indice sonoro 0. |



Ogni byte, checksum incluso, va trasmesso con parità pari. Gli esempi sono costruiti dal codice e verificati nel parser C, non catturati da un veicolo. Per collegarsi alla linea serve il corretto adattamento elettrico previsto dal transceiver LIN.


---



## Copertura, modalità e riferimenti


La configurazione caricata seleziona la **modalità 2**. Il dispatcher riconosce anche modalità 4 e 6: i tipi specifici cambiano da 2/3 a 4/5 e 6/7; per il gruppo 0 i LED vengono letti da D1 anziché D0. La modalità 4 usa D3 per il suono, le modalità 2 e 6 D2. I tipi 1 e D/E/F hanno percorsi comuni con eccezioni specifiche.

Non sono stati attribuiti significati fisici certi ai gruppi 1, 2 e 3: il rapporto documenta lunghezze e destinazioni RAM, senza assegnare per supposizione comandi di temperatura, diagnostica, distanze o luminosità. La presenza di routine di trasmissione nel firmware non prova quali risposte la centralina o questa specifica scheda scambino durante l’uso reale.


## Esempio di verifica del checksum e della parità


Per C1 34 00 55 00:
**0xC1 + 0x34 + 0x00 + 0x55 + 0x00 = 0x14A**
Il byte finale è quindi **0x4A**. Il bit di parità di ciascun byte vale 1 quando gli otto bit dati contengono un numero dispari di “1”, così il totale dati più parità diventa pari.


## Riscontri disponibili


| Elemento | Indirizzi parola PIC / verifica |

| --- | --- |

| Configurazione UART | 0x14D4…0x14DF: SPBRG=63, BRGH=1, SYNC=0, RX9/TX9 attivi. |

| Parità e parser | 0x0421…0x0437 e 0x0438…0x04CA. |

| Dispatcher dei comandi | 0x021E…0x02F6; configurazione EEPROM a 0x0698…0x06DA. |

| Applicazione gruppo 0 | 0x08CE…0x0A18. |

| Mappatura LED / livelli | 0x1FAA…0x1FFF e tabella 0x1822…0x1831. |

| Conteggi sonori | Tabella 0x1801…0x1820. |

| Integrità dell’assembly | Riassemblaggio identico a tutti i 16.906 byte presenti nell’HEX originale. |

| Confronti delle routine | 9.949 casi: parità, parser, dispatcher, mapping LED, tick luminosità, EEPROM, tabelle e percorso di gruppo 0. |




## Limiti


Le verifiche sono statiche e mediante un modello delle istruzioni PIC. Non sono state eseguite prove sulla scheda, registrazioni della seriale o misure del cicalino. I conteggi di prova non equivalgono a una dimostrazione formale dell’intero firmware.

Il C precedentemente consegnato è **parziale**: non è un firmware XC8 completo pronto da programmare. Il parser C semplifica coda e scheduler, e la lettura proposta del nono bit precede RCREG, mentre l’ISR originale lo legge dopo. Il comportamento della FIFO, degli errori UART, degli interrupt e dei feedback analogici richiede ulteriori verifiche.

“Non utilizzato” si riferisce al percorso e alla modalità indicati, non necessariamente all’intero protocollo o al veicolo. Non sono note l’identità della centralina trasmittente, la scala in centimetri, la cadenza reale dei pacchetti o i nomi ufficiali Mercedes dei campi.

Firmware: PIC16F917_benz2.hex. SHA-256 dell’allegato originale: df4fc64997e730b548eb8838211c88c877440bd9c2d7fc9ec5b3ce7f42f17530. Fonti: HEX e schema forniti dall’utente, disassemblazione gputils 1.5.2, codice C e risultati delle prove conservati nell’ambiente.
