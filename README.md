flowchart TB

%% ===============================
%% LAYER 1 — SOURCE SYSTEMS
%% ===============================

subgraph SOURCE["Source Systems (Microsoft 365)"]

    DYN["Dynamics 365 (Dataverse)<br/>• Record Ownership<br/>• systemusers Entity"]

    TEAMS["Microsoft Teams<br/>• Team Membership"]

    AAD["Azure AD (Entra ID)<br/>• Users<br/>• Groups<br/>• Authentication"]

    SP["SharePoint Online<br/>• Record Folders<br/>• Permission ACL"]

end

GRAPH["Microsoft Graph API<br/>Unified Communication Gateway"]

%% ===============================
%% LAYER 2 — AZURE BACKEND
%% ===============================

subgraph AZURE["Azure Environment"]

    APP["Azure App Service<br/>Security Orchestrator"]

    subgraph COMPONENTS["Backend Components"]

        SUB["Subscription Manager<br/>• Create Subscription<br/>• Renew Subscription"]

        WEB["Webhook Controller<br/>• Receive Notifications<br/>• Validate Requests"]

        AUTH["Authentication Module<br/>• Acquire Token<br/>• Cache Token"]

        BL["Business Logic Engine<br/>• Detect Change Type<br/>• Identity Mapping<br/>• Permission Comparison"]

        SPCTRL["SharePoint Permission Controller<br/>• Fetch Current ACL<br/>• Add/Remove Users"]

        LOG["Logging & Audit Module"]

    end

end

%% ===============================
%% CHANGE FLOW
%% ===============================

DYN -- Owner Updated --> GRAPH
TEAMS -- Member Added/Removed --> AAD
AAD -- Group Membership Updated --> GRAPH

SUB -- Register Subscription --> GRAPH
GRAPH -- Webhook Notification --> WEB

WEB --> AUTH
AUTH --> GRAPH

WEB --> BL

BL -- Fetch Record Data --> GRAPH
BL -- Fetch Group Members --> GRAPH
GRAPH -- Return Azure AD Identities --> BL

BL --> SPCTRL
SPCTRL -- Get Current Permissions --> GRAPH
SPCTRL -- Apply Permission Updates --> GRAPH
GRAPH --> SP

SP --> LOG
BL --> LOG
