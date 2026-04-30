# Role-Based Access Control (RBAC) - Azure DevOps

Questo documento definisce la strategia di gestione degli accessi e la convenzione dei nomi (Naming Convention) per i gruppi di sicurezza Active Directory associati ad Azure DevOps.

## Naming Convention

La nomenclatura dei gruppi AD segue questo standard:
`[Prefisso Aziendale]_[Servizio]_[Ruolo]_[Progetto]`

- **Prefisso Aziendale**: `LTS`
- **Servizio**: `AZDO` (Azure DevOps)
- **Ruolo**: `Developers`, `Analysts`, `BuildAdmins`, `ReleaseAdmins`, `Deploy_<Stage>`, `Admins`
- **Progetto**: Nome del progetto specifico (es. `Dynacos`, `LCS`, `SA`). *Nota: il progetto è omesso per i ruoli di governance globale o cross-project.*

**Esempio**: `LTS_AZDO_Developers_Dynacos`

---

## Modello di Sicurezza Ibrido

L'architettura dei permessi si basa su un modello ibrido:
1. **Segregazione Orizzontale (Per Progetto)**: I ruoli operativi di base (sviluppo, analisi, gestione CI/CD) sono isolati per evitare accessi non autorizzati e commit errati tra progetti diversi.
2. **Centralizzazione Verticale (Cross-Project)**: I colli di bottiglia decisionali e amministrativi (approvazioni di deploy, amministrazione globale) sono unificati per facilitare la governance aziendale.

---

## Matrici dei Permessi per Progetto (Segregati)

### 🔷 Progetto: Dynacos
| Ruolo | Gruppo AD | Gruppo DevOps assegnato | Note operative |
| :--- | :--- | :--- | :--- |
| **Developers** | `LTS_AZDO_Developers_Dynacos` | Contributors | Codice, PR, build, Work Items |
| **Analisti** | `LTS_AZDO_Analysts_Dynacos` | Readers | + Edit Work Items (Area Path Dynacos) |
| **Build Admin** | `LTS_AZDO_BuildAdmins_Dynacos` | Build Administrators | Solo build pipeline Dynacos |
| **Release Admin** | `LTS_AZDO_ReleaseAdmins_Dynacos` | Release Administrators | Solo release Dynacos |

### 🔷 Progetto: LCS
| Ruolo | Gruppo AD | Gruppo DevOps assegnato | Note operative |
| :--- | :--- | :--- | :--- |
| **Developers** | `LTS_AZDO_Developers_LCS` | Contributors | Codice, PR, build, Work Items |
| **Analisti** | `LTS_AZDO_Analysts_LCS` | Readers | + Edit Work Items (Area Path LCS) |
| **Build Admin** | `LTS_AZDO_BuildAdmins_LCS` | Build Administrators | Solo build pipeline LCS |
| **Release Admin** | `LTS_AZDO_ReleaseAdmins_LCS` | Release Administrators | Solo release LCS |

### 🔷 Progetto: SA
| Ruolo | Gruppo AD | Gruppo DevOps assegnato | Note operative |
| :--- | :--- | :--- | :--- |
| **Developers** | `LTS_AZDO_Developers_SA` | Contributors | Codice, PR, build, Work Items |
| **Analisti** | `LTS_AZDO_Analysts_SA` | Readers | + Edit Work Items (Area Path SA) |
| **Build Admin** | `LTS_AZDO_BuildAdmins_SA` | Build Administrators | Solo build pipeline SA |
| **Release Admin** | `LTS_AZDO_ReleaseAdmins_SA` | Release Administrators | Solo release SA |

---

## Ruoli Centralizzati (Cross-Project)

Questi ruoli gestiscono le operation comuni e l'amministrazione su tutti i progetti dell'istanza. 
Vengono mappati nel gruppo `Project Valid Users` dei vari progetti solo per "esistere" a livello di progetto senza ottenere permessi impliciti sui repository. I loro privilegi reali vengono applicati tramite le policy e i check sugli Environment.

| Ruolo | Gruppo AD | Gruppo DevOps assegnato | Note operative |
| :--- | :--- | :--- | :--- |
| **Deploy QA** | `LTS_AZDO_Deploy_QA` | Project Valid Users | + Approver stage QA |
| **Deploy PREP** | `LTS_AZDO_Deploy_Prep` | Project Valid Users | + Approver stage PREP |
| **Deploy PROD** | `LTS_AZDO_Deploy_Prod` | Project Valid Users | + Approver stage PROD |
| **Admin** | `LTS_AZDO_Admins` | Project Administrators | Full control sull'intera istanza |
