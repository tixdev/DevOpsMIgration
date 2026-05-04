# Role-Based Access Control (RBAC) - Azure DevOps

This document defines the access management strategy and the Naming Convention for Active Directory security groups associated with Azure DevOps.

## Naming Convention

AD group nomenclature follows this standard:
`[Company Prefix]_[Service]_[Role]_[Project]`

- **Company Prefix**: `LTF`
- **Service**: `AZDO` (Azure DevOps)
- **Role**: `Developers`, `Analysts`, `BuildAdmins`, `ReleaseAdmins`, `ReleaseReviewers`, `ExecDeploy_<Stage>`, `ApproveDeploy_<Stage>`, `ReleaseMgmt`, `Admins`
- **Project**: Name of the specific project (e.g., `Dynacos`, `LCS`, `SA`). *Note: the project is omitted for global or cross-project governance roles.*

**Example**: `LTF_AZDO_Developers_Dynacos`

---

## Hybrid Security Model

The permissions architecture is based on a hybrid model:
1. **Horizontal Segregation (Per Project)**: Basic operational roles (development, analysis, CI/CD management) are isolated to prevent unauthorized access and erroneous commits across different projects.
2. **Vertical Centralization (Cross-Project)**: Decision-making and administrative bottlenecks (deployment approvals, global administration) are unified to facilitate corporate governance.

---

## Permission Matrices per Project (Segregated)

### 🔷 Project: Dynacos
| Role | AD Group | Assigned DevOps Group | Operational Notes |
| :--- | :--- | :--- | :--- |
| **Developers** | `LTF_AZDO_Developers_Dynacos` | Contributors | Code, PRs, builds, Work Items |
| **Analysts** | `LTF_AZDO_Analysts_Dynacos` | Readers | + Edit Work Items (Dynacos Area Path) |
| **Build Admin** | `LTF_AZDO_BuildAdmins_Dynacos` | Build Administrators | Dynacos build pipelines only |
| **Release Admin** | `LTF_AZDO_ReleaseAdmins_Dynacos` | Release Administrators | Dynacos releases only |
| **Release Reviewer** | `LTF_AZDO_ReleaseReviewers_Dynacos` | Contributors | Mandatory reviewer for Release PRs |

### 🔷 Project: LCS
| Role | AD Group | Assigned DevOps Group | Operational Notes |
| :--- | :--- | :--- | :--- |
| **Developers** | `LTF_AZDO_Developers_LCS` | Contributors | Code, PRs, builds, Work Items |
| **Analysts** | `LTF_AZDO_Analysts_LCS` | Readers | + Edit Work Items (LCS Area Path) |
| **Build Admin** | `LTF_AZDO_BuildAdmins_LCS` | Build Administrators | LCS build pipelines only |
| **Release Admin** | `LTF_AZDO_ReleaseAdmins_LCS` | Release Administrators | LCS releases only |

### 🔷 Project: SA
| Role | AD Group | Assigned DevOps Group | Operational Notes |
| :--- | :--- | :--- | :--- |
| **Developers** | `LTF_AZDO_Developers_SA` | Contributors | Code, PRs, builds, Work Items |
| **Analysts** | `LTF_AZDO_Analysts_SA` | Readers | + Edit Work Items (SA Area Path) |
| **Build Admin** | `LTF_AZDO_BuildAdmins_SA` | Build Administrators | SA build pipelines only |
| **Release Admin** | `LTF_AZDO_ReleaseAdmins_SA` | Release Administrators | SA releases only |

---

## Centralized Roles (Cross-Project)

These roles manage common operations and administration across all projects on the instance.
They are mapped into the `Project Valid Users` group of the various projects solely to "exist" at the project level without gaining implicit permissions on the repositories. Their actual privileges are enforced via Environment checks and policies.

| Role | AD Group | Assigned DevOps Group | Operational Notes |
| :--- | :--- | :--- | :--- |
| **Deploy QA** | `LTF_AZDO_ExecDeploy_QA` | Project Valid Users | + Approver stage QA |
| **Deploy PREP** | `LTF_AZDO_ExecDeploy_Prep` | Project Valid Users | + Approver stage PREP |
| **Release Management** | `LTF_AZDO_ReleaseMgmt` | Project Valid Users | **1st** Approver stage PROD (Sequential) |
| **Approve PROD** | `LTF_AZDO_ApproveDeploy_Prod` | Project Valid Users | **2nd** Approver stage PROD (Sequential) |
| **Exec PROD** | `LTF_AZDO_ExecDeploy_Prod` | Project Valid Users | **3rd** Approver stage PROD (Sequential) - **Execution Trigger** |
| **Admin** | `LTF_AZDO_Admins` | Project Administrators | Full control over the entire instance |
