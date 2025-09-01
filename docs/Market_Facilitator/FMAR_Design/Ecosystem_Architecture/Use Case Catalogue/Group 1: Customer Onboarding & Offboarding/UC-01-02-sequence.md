%% UC-01.02: Register Organisation
sequenceDiagram
autonumber
  actor Admin as Org Admin (Authorised Rep)
  participant UI as FMAR UI/API (SPUM)
  participant CH as Company Registry (e.g., CH API)
  participant RC as Role Catalogue
  participant Mail as Email / Notify
  participant Audit as Audit Log

  Admin->>UI: Submit organisation {legalName, regNo, roles[], contact}
  UI->>UI: Check uniqueness(regNo/VAT, legalName)

  alt duplicate found
    UI-->>Admin: Organisation already exists (existingOrgId)
  else unique
    UI->>CH: Verify company(regNo)/status
    CH-->>UI: {exists, active}

    alt invalid or inactive
      UI-->>Admin: Registration rejected (invalid/inactive company)
    else valid
      UI->>UI: CreateOrganisation() -> orgId

      loop for each selected role
        UI->>RC: GetBaselinePermissions(role)
        RC-->>UI: BaselinePermissions
        UI->>UI: AssignRole(orgId, role, permissions)
      end

      UI->>Audit: Record(OrganisationCreated, orgId, roles[])
      UI->>Mail: Send confirmation(orgId, summary)
      UI-->>Admin: Organisation registered (orgId, rolesAssigned[])
    end
  end
