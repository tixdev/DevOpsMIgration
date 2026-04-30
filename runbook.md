# Piano di Migrazione Azure DevOps Server

Di seguito le macro-fasi per la migrazione cross-domain, strutturate in ordine **cronologico**, separando ciò che può essere svolto in anticipo dalle attività vincolate al momento del downtime.

---

## 1. Attività Preparatorie (Giorni Precedenti al Downtime)

Questa fase non impatta l'operatività corrente e serve ad allestire l'ambiente nel nuovo dominio per farsi trovare pronti il giorno della migrazione.

*   **Provisioning Infrastruttura:** 
    *   Creazione dei nuovi server virtuali (Azure DevOps e SQL Server).
    *   Installazione dei prerequisiti OS (IIS, framework, database engine).
    *   Provisioning delle nuove macchine (OS level) che ospiteranno i nuovi Build Agent.
*   **Gestione Identità e Gruppi AD:** 
    *   Creazione dei nuovi account di servizio nel nuovo dominio (es. `AZDO_SVC_Agent`).
    *   Creazione dei nuovi Gruppi di Sicurezza AD e popolamento degli stessi con le utenze aziendali (vedi *Appendice*).
*   **Pianificazione Downtime:** 
    *   Concordare e comunicare in anticipo la finestra di "Downtime" ai team di sviluppo.

---

## 2. Finestra di Migrazione (Day 0 - Inizio Downtime)

Le attività seguenti comportano l'interruzione del servizio per gli sviluppatori e il "congelamento" dei dati.

*   **Freeze del Vecchio Sistema:**
    *   Comunicazione formale di inizio downtime.
    *   Messa in quiescence del server DevOps esistente.
    *   Arresto dei servizi DevOps e degli application pool IIS per impedire qualsiasi nuova connessione o modifica al codice/ticket.
*   **Migrazione dei Dati (Database):**
    *   Esecuzione del backup completo (Full Backup) del database di configurazione (`Tfs_Configuration`) e di tutte le Collection presenti nel vecchio SQL.
    *   Trasferimento dei file di backup sul nuovo ambiente.
    *   Ripristino dei database sulla nuova istanza SQL Server.
*   **Setup del Nuovo Azure DevOps:**
    *   Installazione dei binari di DevOps sul nuovo server.
    *   Esecuzione del wizard di configurazione scegliendo la modalità "Application Tier Only", e puntando ai database appena ripristinati.
    *   Impostazione della nuova Public URL del servizio.

---

## 3. Riconfigurazione Applicativa (Day 0 - Durante il Downtime)

Poiché abbiamo scelto di non migrare le vecchie identità ("Da Zero"), le configurazioni di sicurezza e gli agenti vanno ricollegati sul nuovo server prima di riaprire gli accessi.

*   **Sicurezza e Accessi:**
    *   Accesso alla nuova console DevOps con l'account amministratore.
    *   Ripopolamento rapido dei gruppi di sicurezza di DevOps agganciandoli ai gruppi AD pre-creati nella fase 1 (es. inserendo il gruppo AD `AZDO_Developers` nel gruppo DevOps `Contributors`).
*   **Riconfigurazione CI/CD (Agenti):**
    *   Creazione dei nuovi Agent Pool all'interno di DevOps.
    *   Generazione dei Personal Access Token (PAT) amministrativi.
    *   Sulle macchine agent (preparate nella fase 1), download del client, installazione e registrazione collegandoli ai nuovi Pool.

---

## 4. Collaudo e Go-Live (Day 0/1 - Fine Downtime)

Attività finali di verifica per ripristinare l'operatività aziendale in sicurezza.

*   **Collaudo Tecnico (Smoke Test):**
    *   Verifica che le utenze AD create in fase 1 riescano a loggarsi.
    *   Test di visibilità dei repository e delle board.
    *   **Verifica Ambiente di Sviluppo (VDI):** Dalle nuove macchine di sviluppo (VDI), effettuare il git clone del repository `dynacos`, lanciare una build in locale e avviare l'applicativo per validare il workflow dello sviluppatore (autenticazione git, scaricamento pacchetti e compilazione).
    *   Esecuzione manuale di una pipeline **mono (legacy)** per confermare che i nuovi agent comunichino correttamente con il server DevOps. *(Nota: il collaudo delle pipeline relative alle API sarà eseguibile soltanto al completamento della Fase 5).*
*   **Apertura del Servizio:**
    *   Comunicazione ufficiale di fine downtime a tutti i team, condividendo la nuova Public URL.

---

## 5. Attività Post Go-Live

Dopo aver ripristinato l'operatività principale di Azure DevOps, è necessario adeguare le pipeline e i repository esterni che gestiscono le configurazioni di deployment per allinearli alla nuova infrastruttura.

*   **Migrazione Repository Configurazioni di Deployment:**
    *   **Clonazione (Sorgente):** Recupero dei repository dal server storico `git.corner.ch` (attualmente sotto l'ambiente *restricted*).
    *   **Rinominazione Progetto:** Cambio logico e fisico del namespace/progetto da `win_config` (vecchio) a `sys_conf` (nuovo).
    *   **Push (Destinazione):** Caricamento dei repository migrati sul nuovo server Bitbucket associato all'ambiente *ittest*.
    *   **Adeguamento Script Cake:** Modifica dei file `build.cake` (o script di deploy Cake correlati) affinché le operazioni di push e tagging automatico avvengano sui nuovi repository Bitbucket anziché su quelli vecchi.
    *   **Aggiornamento Azure DevOps:** Modifica delle *Service Connection* o dei task YAML all'interno delle Release Pipeline affinché peschino i file di configurazione (`sys_conf`) dal nuovo repository Bitbucket (*ittest*) anziché dal vecchio nodo restricted.
    *   **Collaudo Pipeline API:** A questo punto, avviare l'esecuzione manuale delle pipeline API per confermare il successo della migrazione dei repo di configurazione e la corretta esecuzione del tagging via Cake.

---

## Appendice: Gestione Permessi (RBAC)

Per mantenere il runbook puramente operativo, tutte le logiche di profilazione, la *Naming Convention* dei gruppi AD (`LTS_AZDO_*`) e le specifiche matrici dei permessi suddivise per progetto sono state estratte e documentate nel file separato:

👉 **[Consulta il documento RBAC](file:///Users/tizianocrisci/DevOpsMIgration/rbac.md)**

I gruppi AD elencati nel documento RBAC **devono essere creati e popolati su Active Directory durante la Fase 1**, in modo da essere pronti all'uso per la mappatura prevista in Fase 3.
