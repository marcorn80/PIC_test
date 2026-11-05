# PIC16F917 PDC Mercedes

[Scarica i sorgenti e la documentazione](PDC_Mercedes_sorgenti.zip).

Il pacchetto contiene la ricostruzione assembly completa, verificata byte per byte rispetto al file HEX fornito, e una traduzione C **parziale**, con moduli per integrazione XC8. Il C non è un firmware completo pronto da programmare e non è stato compilato con XC8 né verificato sulla scheda.

Estrarre lo ZIP e leggere `PDC_Mercedes/README.md` e `PDC_Mercedes/ANALISI.md` per copertura, verifiche e limitazioni.

SHA-256 dello ZIP: `8deb94a50a85f5d531a86de1a85ef7ddc54bab27cdc14cad6d9bc6d21796a681`.

## Display Skoda Superb — PIC16F914

[Scarica il progetto C per XC8](Skoda_Superb_PIC16F914_XC8.zip).

Include main, driver MCP2515 a 100 kbit/s in listen-only, LCD, orologio, temperatura, retroilluminazione ID 0x635 e standby. Verifiche offline: 7.168 casi di confronto delle routine, scenario applicativo e 36 scritture MCP2515 corrispondenti all’originale. **Compilazione XC8, occupazione flash/RAM/stack e prova sulla scheda ancora non verificate:** il compilatore non è installato e il download è bloccato dalla rete. Non è un HEX pronto da programmare. Leggere il README nello ZIP per istruzioni, limiti e differenze rispetto al binario.
