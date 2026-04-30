# Bozza Comunicazione: Annuncio Migrazione

**Oggetto:** [IMPORTANTE] Transizione al Nuovo Ambiente di Sviluppo (VDI) e Migrazione DevOps - Venerdì 15 Maggio ore 17:00

Ciao Team,

vi informiamo che **Venerdì 15 Maggio a partire dalle ore 17:00** prenderà il via il passaggio ufficiale verso il nostro nuovo ecosistema di sviluppo (VDI). Come tassello fondamentale di questa transizione globale, l'attuale server di Azure DevOps in ambiente *restricted* verrà spento per essere migrato sulla nuova infrastruttura.

**Cosa comporta:**
* **Cambio Ambiente:** Al termine delle operazioni, l'intero ciclo di sviluppo si sposterà sulle nuove macchine virtuali (VDI).
* **Blocco Operatività (Downtime DevOps):** A partire dalle 17:00 del 15 Maggio, il servizio Azure DevOps sarà completamente congelato. Non sarà possibile effettuare push di codice sui repository, accedere alle board o lanciare le pipeline di CI/CD.
* Vi consigliamo vivamente di effettuare eventuali commit locali e completare i deployment urgenti con largo anticipo rispetto a questa finestra temporale.

Le attività tecniche di migrazione infrastrutturale si svolgeranno nel corso del weekend.

**Documentazione e Risorse Utili:**
Per maggiore trasparenza e per aiutarvi a familiarizzare con il nuovo ambiente, vi condividiamo i seguenti documenti tecnici:
1. **Procedura Setup VDI di Sviluppo:** [Inserire Link alla procedura VDI]
2. **Runbook Operativo di Migrazione:** [Inserire Link al documento Runbook]
3. **Modello Gestione Permessi (RBAC):** [Inserire Link al documento RBAC]

**Importante - Verifica Setup VDI:**
La procedura di setup delle VDI è disponibile ormai da diverse settimane proprio per consentire a ciascuno di gestire la configurazione e risolvere eventuali dubbi con calma. Vi invitiamo caldamente a completare i test di funzionamento il prima possibile: non aspettate lunedì per il primo accesso, così da garantire la vostra piena operatività fin dal momento della riapertura e prevenire ritardi nelle attività di sviluppo che non dipendano dall'infrastruttura.

**Prossimi Passi:**
Al termine delle attività riceverete una mail di follow-up. Questa comunicazione confermerà la corretta conclusione della migrazione (fornendovi le nuove istruzioni di accesso e la nuova URL) oppure, nell'eventualità di problemi critici, vi avviserà dell'avvenuto rollback al vecchio ambiente.

Grazie per la collaborazione e per la pazienza.

Cordiali saluti,

[Il Tuo Nome / Il Team DevOps]
