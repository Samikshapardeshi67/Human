flowchart LR

    subgraph Microsoft_365
        D[ Dynamics 365 <br> (Dataverse Records) ]
        T[ Microsoft Teams <br> (Team Membership) ]
        AAD[ Azure AD <br> (Users & Groups) ]
        SP[ SharePoint Online <br> (Folders & Permissions) ]
    end

    G[ Microsoft Graph API ]
    BE[ Azure App Service <br> Security Orchestrator Backend ]

    %% Change Detection
    D -- Record Ownership Change --> G
    T -- Team Membership Change --> AAD
    AAD -- Group Membership Update --> G

    %% Subscription Flow
    BE -- Create Subscription --> G
    G -- Webhook Notification --> BE

    %% Data Fetching
    BE -- Fetch Updated Data --> G
    G -- Returns Users / Owners --> BE

    %% SharePoint Update
    BE -- Update Folder Permissions --> G
    G -- Apply Permission Changes --> SP
