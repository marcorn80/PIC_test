Display Skoda Superb
Mappa dei messaggi CAN

PIC16F914 · MCP2515 · TLE6254 | Revisione 1 · 5 novembre 2026

Descrizione ricavata dal firmware fornito dall’utente. Gli identificatori riportati sono **ID CAN standard a 11 bit** (spesso chiamati informalmente “PID”); non sono PID diagnostici OBD-II.

| Parametro | Valore / convenzione |

| --- | --- |

| Rete | CAN low-speed / fault-tolerant; front-end TLE6254 confermato dall’utente. |

| Velocità | 100 kbit/s con quarzo MCP2515 da 8 MHz; CNF1=81, CNF2=BE, CNF3=83 (esadecimali), sample point 80%. |

| Direzione | Solo ricezione: TXCAN scollegato e MCP2515 in listen-only. Il display non invia ACK. |

| Byte e bit | D0 = primo byte dati, D7 = ottavo. Bit 0 = meno significativo; bit 7 = più significativo. |

| DLC | Esempi e port C usano 8 byte. Il binario originale non verifica sistematicamente la lunghezza. |




## 0x65D — Orario


| Campo | Significato osservato nel display |

| --- | --- |

| D0…D4 | Non utilizzati nel percorso di visualizzazione dell’orario analizzato. |

| D5, bit 7…4 | Quattro bit meno significativi delle ore; D5 bit 3…0 ignorati. |

| D6, bit 0 | Quinto bit delle ore (peso 16). |

| D6, bit 6…1 | Minuti in binario; D6 bit 7 ignorato dal ricevitore applicativo. |

| D7 | Non utilizzato per questo messaggio. |



**Ore = (D5 &gt;&gt; 4) | ((D6 &amp; 1) &lt;&lt; 4)**
**Minuti = (D6 &gt;&gt; 1) &amp; 0x3F**

Sono campi binari, non BCD. Il formato 12/24 ore dipende da 0x60E. Se 0x65D non viene ricevuto per 154 tick (circa 10,09 s nominali), il firmware marca l’orario come non disponibile. I range codificabili 0…31 e 0…63 non garantiscono che valori fuori dall’orario civile siano validi.


## 0x527 — Temperatura


| Campo | Significato osservato nel display |

| --- | --- |

| D0…D4 | Non utilizzati per la temperatura. |

| D5 | Temperatura codificata: T[°C] = D5 / 2 − 50. Risoluzione del campo: 0,5 °C. |

| D6 | Non utilizzato per la temperatura. |

| D7, bit 0 | 0: nessuna segnalazione di indisponibilità; 1: dato non disponibile. Gli altri bit sono ignorati. |



D5=0xFF è trattato come non valido anche se D7 bit 0 è zero. D5=0x64 → 0 °C; 0x96 → 25 °C; 0x50 → −10 °C. Dopo 154 tick senza il messaggio viene forzata l’indicazione non disponibile. La temperatura numerica mostrata segue gli arrotondamenti del renderer; la formula descrive il valore fisico codificato.


---



## 0x635 — Retroilluminazione del display


Il collegamento alla luminosità è verificato nel codice: i campi ricevuti alimentano la routine che aggiorna il **PWM a 10 bit su RC5/CCP1**, connesso al circuito di retroilluminazione. Il ricalcolo avviene quando D0 o D2 cambiano, se la visualizzazione è attiva.

| Campo | Significato / trattamento |

| --- | --- |

| D0 | Ingresso del livello di regolazione. 0 viene normalizzato a 100; 20…100 restano invariati; gli altri valori sono normalizzati a 20. Non equivale direttamente al duty PWM. |

| D1 | Non utilizzato dal percorso analizzato. |

| D2 | Ingresso della curva: 10…253 accettati; valori <10 oppure 254/255 trattati come 10. La routine sottrae poi 10 e interpola nelle tabelle. |

| D3…D7 | Non utilizzati dal percorso analizzato. |



La routine combina D0 e D2 con soglie e interpolazione non lineare. Dopo **20 tick senza 0x635** (circa 1,31 s nominali), ricalcola il PWM usando 100 per il livello normalizzato; D2 continua a partecipare al calcolo. Non significa necessariamente “PWM al 100%”.

Resta sconosciuto cosa rappresenti D2 nel veicolo: per esempio luce ambiente, compensazione o altro parametro. Il binario prova il suo uso nel calcolo, non l’origine fisica del segnale.


## 0x60E — Unità temperatura e formato ora


| Campo | Significato |

| --- | --- |

| D0, bit 1 (maschera 0x02) | 0: Celsius; 1: Fahrenheit. |

| D0, bit 7 (maschera 0x80) | 0: formato 24 ore; 1: formato 12 ore. |

| Altri bit / D1…D7 | Non utilizzati da questo display nel percorso analizzato. |



| D0 | Impostazione |

| --- | --- |

| 00 | Celsius / 24 ore |

| 02 | Fahrenheit / 24 ore |

| 80 | Celsius / 12 ore |

| 82 | Fahrenheit / 12 ore |



Le impostazioni cambiano il rendering; non cambiano la codifica CAN di D5 nel messaggio 0x527 o dei campi orario in 0x65D.


## 0x575 — Consenso alla visualizzazione


| Campo | Significato |

| --- | --- |

| D0, bit 1 (maschera 0x02) | 1: richiesta di visualizzazione attiva; 0: richiesta non attiva. |

| Altri bit / D1…D7 | Non utilizzati da questo display nel percorso analizzato. |



Sul passaggio ad attivo il firmware ridisegna ora e temperatura e aggiorna il PWM. Il bit a zero modifica lo stato applicativo; oscuramento e standby sono coordinati anche dalla logica di inattività bus. Non va quindi interpretato come una garanzia di spegnimento fisico istantaneo.


---



## Esempi di pacchetti


**Esempi sintetici**, costruiti dalle formule ricostruite; non sono registrazioni di traffico reale. Tutti i valori nel payload sono esadecimali. Nella notazione ID#DATI, il payload dopo # contiene 8 byte.

| Pacchetto | Interpretazione |

| --- | --- |

| 65D#0000000000C04400 | Ore 12, minuti 34 → 12:34. |

| 65D#0000000000707700 | Ore 23, minuti 59 → 23:59 (formato 24 ore). |

| 527#0000000000960000 | D5=0x96 → 25 °C; flag indisponibilità assente. |

| 527#0000000000500000 | D5=0x50 → −10 °C; flag indisponibilità assente. |

| 527#0000000000960001 | D7 bit 0 = 1 → temperatura non disponibile. |

| 60E#0000000000000000 | Celsius e formato 24 ore. |

| 60E#8200000000000000 | Fahrenheit e formato 12 ore. |

| 635#3200800000000000 | D0=50, D2=128: ingressi per la curva PWM, non duty 50%. |

| 575#0200000000000000 | Richiesta di visualizzazione attiva. |




## Uso e limiti della ricostruzione


Il formato dei frame e il significato attribuito ai campi descrivono **ciò che questo display utilizza**. “Non utilizzato” non significa che il byte non abbia altre funzioni sul bus del veicolo. Non sono state determinate le centraline sorgenti, le periodicità reali dei messaggi o un eventuale significato ufficiale Skoda/VAG.

La scheda è di sola ricezione e non fornisce ACK. Per eventuali prove al banco occorrono un’interfaccia elettrica compatibile con il CAN low-speed/fault-tolerant e un trasmettitore che gestisca l’assenza di ACK, oppure un secondo nodo che lo fornisca. Il solo bitrate di 100 kbit/s non basta a garantire compatibilità elettrica.

I timeout sono calcolati dal Timer1 del PIC interno a 8 MHz: un tick nominale vale circa 65,536 ms. Latenza e accorpamento dei tick possono modificare i tempi effettivi. Non sono state eseguite misure sulla scheda o catture CAN.


## Riferimenti verificabili


| Elemento | Posizione nel firmware (indirizzi parola PIC) |

| --- | --- |

| Bit timing MCP2515 | 0x0960…0x0975 |

| Ricezione / ID standard | Routine 0x06F4; ID = (SIDH << 3) | (SIDL >> 5) |

| Orario / temperatura | Dispatch 0x0842…0x08A7; renderer 0x00EC…0x0698 |

| 0x635 e PWM | Dispatch 0x0A82…0x0AB5; curva 0x0B5D…0x0D69 |

| 0x60E / 0x575 | Dispatch 0x0AB6…0x0B0B |

| Timeout | 0x08C4…0x08F4 |



Firmware analizzato: display_Skoda.hex. SHA-256 dell’allegato: 3810718c5f49c0bda5b47ece34e9076434c1a0cbcaa6053a544d577bfbf735d0. Le routine C corrispondenti hanno superato 7.168 confronti con un modello delle istruzioni originali; ciò non sostituisce una verifica sul dispositivo.
