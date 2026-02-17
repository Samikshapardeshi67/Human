Problem Statement
In Microsoft 365 environments, organizations use multiple services such as:
Dynamics 365 (Dataverse)
Microsoft Teams
Azure AD Groups
SharePoint Online
Each of these systems manages users and permissions independently.
However, SharePoint document folders often need to reflect security defined in:
Dynamics record ownership
Team membership
Azure AD group membership
Currently:
Security changes in one system do not automatically reflect in SharePoint.
Manual updates cause inconsistency.
This creates security risks and compliance issues.
Therefore, a centralized system is required to synchronize security changes across Microsoft services and enforce consistent SharePoint permissions.
2️⃣ Objective of the System
Build a Central Security Orchestrator that:
Detects security-related changes in Dynamics, Teams, and Azure AD.
Fetches updated access information.
Updates SharePoint folder permissions accordingly.
Maintains consistent access control across systems.
3️⃣ High-Level Architecture
The system contains:
Azure AD (Identity Provider)
Microsoft Graph API (Unified Gateway)
Azure App Service (Backend Engine)
SharePoint Online (Target System)
Dynamics / Teams / Azure AD (Source Systems)
Architecture flow:
Security Change in Microsoft App
↓
Microsoft Graph detects change
↓
Graph sends notification (webhook)
↓
Azure Backend processes event
↓
Backend fetches full data from Graph
↓
Backend updates SharePoint permissions
↓
Log result
4️⃣ Step-by-Step Implementation Process
Now we go step-by-step from system creation.
Step 1 — Register Application in Azure AD
Purpose: Authentication
You create an App Registration in Azure AD.
You receive:
Client ID
Tenant ID
Client Secret
You assign API permissions:
Sites.ReadWrite.All
Group.Read.All
User.Read.All
Directory.Read.All
Admin grants consent.
This allows your backend to call Microsoft Graph.
Step 2 — Backend Setup (Azure App Service)
You build backend using:
Node.js OR
.NET Core
Deploy it to Azure App Service.
Backend exposes endpoints like:
POST /webhook
This endpoint will receive Graph notifications.
Step 3 — Obtain Access Token
Whenever backend needs to call Graph:
It sends request to:
https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token�
With:
client_id
client_secret
grant_type=client_credentials
scope=https://graph.microsoft.com/.default�
Azure AD returns:
access_token
Backend uses this token to call Graph.
Step 4 — Create Subscriptions (Event Detection)
Backend creates subscriptions using:
POST https://graph.microsoft.com/v1.0/subscriptions�
Example for Teams membership:
{ "changeType": "updated", "notificationUrl": "https://yourbackend.com/webhook�", "resource": "/groups/{group-id}/members", "expirationDateTime": "2026-02-20T11:23:00Z", "clientState": "secret" }
Graph validates webhook.
Backend must respond to validationToken.
Subscription becomes active.
Now Graph monitors that resource.
Step 5 — Change Occurs in Microsoft System
Example:
User added to Teams group.
Graph detects membership change.
Graph sends notification:
POST https://yourbackend.com/webhook�
Body:
{ "value": [ { "subscriptionId": "abc", "resource": "/groups/{group-id}/members", "changeType": "updated" } ] }
Step 6 — Backend Processes Notification
Important: Notification only says “something changed.”
Backend must:
Read resource path.
Call Graph again to fetch full updated data.
Example:
GET /groups/{group-id}/members
Graph returns complete member list.
Step 7 — Backend Builds Correct Security Model
Backend decides:
These members must have access to specific SharePoint folders.
Example internal model:
Folder: SalesDocs
Allowed Users:
Rahul (read)
Anita (read)
Karan (read)
Step 8 — Update SharePoint
Backend calls Graph:
GET /sites/{site-id}/drives
GET /drive/items/{folder-id}/permissions
Then:
Remove users not in group
Add new users
Update roles
Using:
POST /drive/items/{folder-id}/invite
Graph forwards to SharePoint.
SharePoint updates Access Control List.
Step 9 — Logging
Backend stores:
Event source
Resource ID
Time
Success/Failure
Changes made
This is required for audit and debugging.
Step 10 — Subscription Renewal
Subscriptions expire.
Backend must:
Check expiration time.
Before expiry, send:
PATCH https://graph.microsoft.com/v1.0/subscriptions/{id}�
To extend expiration.
If not renewed → notifications stop.
5️⃣ Complete End-to-End Example
Case: Owner changed in Dynamics.
Dynamics owner updated.
Graph sends Dataverse change notification.
Backend receives webhook.
Backend fetches updated record via Graph.
Backend identifies new owner.
Backend updates SharePoint folder permissions.
Logs success.
Case: User removed from Azure AD group.
Group membership changed.
Graph notifies backend.
Backend fetches updated group members.
Backend removes user from SharePoint folders.
Logs event.
