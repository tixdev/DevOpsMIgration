# Azure DevOps Server Migration Plan

Below are the macro-phases for the cross-domain migration, structured in **chronological** order, separating the preparatory activities from those bound to the downtime window.

---

## 1. Preparatory Activities (Days Prior to Downtime)

This phase does not impact current operations and serves to set up the environment in the new domain, ensuring readiness for migration day.

*   **Infrastructure Provisioning:**
    *   Create the new virtual servers (Azure DevOps and SQL Server).
    *   Install OS prerequisites (IIS, frameworks, database engine).
    *   Provision the new machines (at the OS level) that will host the new Build Agents.
*   **AD Identity and Group Management:**
    *   Create new service accounts in the new domain (e.g., `AZDO_SVC_Agent`).
    *   Create new AD Security Groups and populate them with the appropriate corporate users (see the *Appendix*).
*   **Downtime Planning:**
    *   Agree upon and communicate the "Downtime" window to the development teams in advance.

---

## 2. Migration Window (May 15th at 17:00 - Downtime Start)

The following activities require service interruption for developers and the "freezing" of data. **Downtime is scheduled to begin exactly on May 15th at 17:00.**

*   **Freeze the Old System:**
    *   Send formal communication marking the start of downtime.
    *   Put the existing DevOps server into quiescence mode.
    *   Stop Azure DevOps services and IIS application pools to prevent any new connections or modifications to code/tickets.
*   **Data Migration (Databases):**
    *   Perform a Full Backup of the configuration database (`Tfs_Configuration`) and all Collection databases residing on the old SQL instance.
    *   Transfer the backup files to the new environment.
    *   Restore the databases onto the new SQL Server instance.
*   **New Azure DevOps Setup:**
    *   Install the DevOps binaries on the new server.
    *   Run the configuration wizard, selecting "Application Tier Only" mode, pointing to the newly restored databases.
    *   Set the service's new Public URL.

---

## 3. Application Reconfiguration (Day 0 - During Downtime)

Since we chose not to migrate the old identities ("Clean Slate"), security configurations and agents must be re-linked on the new server before reopening access.

*   **Security and Access:**
    *   Log into the new **Azure DevOps Web Portal (UI)** using the administrator account.
    *   *Why this is necessary:* Although the AD groups were created in Phase 1, Azure DevOps does not automatically know what permissions they should have. You must manually "link" them by navigating to *Project Settings -> Security* and adding the new AD groups into the built-in DevOps security groups (e.g., adding `LTS_AZDO_Developers_Dynacos` into the DevOps `Contributors` group). Once this one-time mapping is done, all future user management is handled entirely via Active Directory.
*   **CI/CD Reconfiguration (Agents):**
    *   Create new Agent Pools within DevOps.
    *   Generate administrative Personal Access Tokens (PAT).
    *   On the agent machines (prepared in Phase 1), download the client, install it, and register the agents by linking them to the new Pools.

---

## 4. Testing and Go-Live (Day 0/1 - End of Downtime)

Final verification activities to safely restore business operations.

*   **Technical Validation (Smoke Test):**
    *   Verify that the AD users created in Phase 1 can log in successfully.
    *   Test the visibility of repositories and boards.
    *   **Development Environment Verification (VDI):** From the new development virtual machines (VDI), perform a git clone of the `dynacos` repository, run a local build, and start the application to validate the developer workflow (git authentication, package restoration, and compilation).
    *   Manually execute a **mono (legacy)** pipeline to confirm that the new agents are communicating correctly with the DevOps server. *(Note: API-related pipeline testing will only be executable upon completion of Phase 5).*
*   **Service Reopening:**
    *   Send official communication marking the end of downtime to all teams, sharing the new Public URL.

---

## 5. Post Go-Live Activities

After restoring the main Azure DevOps operations, it is necessary to adapt the external repositories and pipelines managing deployment configurations to align with the new infrastructure.

*   **Deployment Configurations Repository Migration:**
    *   **Clone (Source):** Retrieve the repositories from the historical server `git.corner.ch` (currently under the *restricted* environment).
    *   **Project Renaming:** Logically and physically change the namespace/project from `win_config` (old) to `sys_conf` (new).
    *   **Push (Destination):** Push the migrated repositories to the new Bitbucket server associated with the *ittest* environment.
    *   **Cake Script Adaptation:** Modify the `build.cake` files (or related Cake deployment scripts) so that automatic push and tagging operations occur on the new Bitbucket repositories rather than the old ones.
    *   **Azure DevOps Update:** Modify the *Service Connections* or YAML tasks within the Release Pipelines so they fetch configuration files (`sys_conf`) from the new Bitbucket repository (*ittest*) instead of the old restricted node.
    *   **API Pipeline Testing:** At this point, manually trigger the API pipelines to confirm the successful migration of the configuration repos and the correct execution of tagging via Cake.

---

## Appendix: Permissions Management (RBAC)

To keep the runbook purely operational, all profiling logic, the AD group *Naming Convention* (`LTS_AZDO_*`), and the specific project-based permission matrices have been extracted and documented in a separate file:

👉 **[View the RBAC document](file:///Users/tizianocrisci/DevOpsMIgration/rbac.md)**

The AD groups listed in the RBAC document **must be created and populated in Active Directory during Phase 1**, so they are ready to be used for mapping during Phase 3.
