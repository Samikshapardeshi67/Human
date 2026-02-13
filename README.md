PROJECT:
Security Synchronization Tool Between Dynamics 365 and SharePoint
🟢 1️⃣ Problem Statement (WHY we need this tool)
In Microsoft ecosystem:
Dynamics 365 (Dataverse) stores records (Accounts, Cases, etc.)
SharePoint stores documents in folders
Security models of both systems are different
They do NOT automatically sync permissions
Example:
In Dynamics:
Account: ABC Pvt Ltd
Owner: Samiksha
Rahul has Read access
But in SharePoint:
Folder may still allow old users
This creates a security risk.
So we build a tool to:
Automatically synchronize Dynamics record security with SharePoint folder permissions.
🟢 2️⃣ Overall Architecture
Copy code

User changes record in Dynamics
        ↓
Trigger fires
        ↓
Backend Service
        ↓
Dataverse API (read security)
        ↓
Microsoft Graph API
        ↓
SharePoint updates folder & permissions
🟢 3️⃣ Environment Setup
Step 1: Create Microsoft 365 Developer Account
WHY? Because you need:
Dynamics 365
SharePoint
Azure AD
Step 2: Register App in Azure AD
WHY? Because your backend must authenticate to call APIs.
After registration, you get:
Client ID
Tenant ID
Client Secret
These are used to generate Access Token.
Without this: ❌ APIs will reject your request.
Step 3: Add API Permissions
For Graph:
Sites.ReadWrite.All
Files.ReadWrite.All
User.Read.All
For Dataverse:
user_impersonation
Then click: ✔ Grant Admin Consent
WHY? Because Microsoft must allow your app to manage SharePoint and read CRM data.
🟢 4️⃣ Trigger Creation
Your backend must run when record changes.
Two options:
Option A: Dataverse Plugin (Professional)
Triggers on:
Create
Update
Assign
Share
Option B: Power Automate (Easy)
Trigger sends:
Copy code

recordId
entityName
To backend.
🟢 5️⃣ Backend Processing Steps
Now real logic begins.
🔹 Step 5.1: Call Dataverse API
WHY? Because trigger does not give full security data.
Call:
Copy code

GET /accounts({recordId})
Fetch:
Owner
Teams
Shared users
Example Response:
Copy code

Owner: samiksha@company.com
Shared: rahul@company.com (Read)
Now backend knows who should have access.
🔹 Step 5.2: Identity Mapping
Problem: Dataverse user ID ≠ Azure AD ID.
SharePoint needs Azure AD identity.
So backend calls Graph:
Copy code

GET /users/samiksha@company.com
Graph returns:
Copy code

id = AAD-999
Now backend has correct Azure AD ID.
WHY? Because SharePoint permission system uses Azure AD.
🔹 Step 5.3: Get Site ID
SharePoint structure:
Tenant → Site → Library → Folder → Files
Example site URL:
Copy code

https://company.sharepoint.com/sites/CRMDocs
Call:
Copy code

GET /sites/company.sharepoint.com:/sites/CRMDocs
Response gives:
Copy code

site-id
Store this permanently in config.
🔹 Step 5.4: Get Document Library (Drive ID)
Call:
Copy code

GET /sites/{site-id}/drives
Select “Documents” library.
Store:
Copy code

drive-id
WHY? Because folders are created inside libraries.
🔹 Step 5.5: Check If Folder Exists
Example folder naming:
Copy code

Account_ABC_12345
Call:
Copy code

GET /drives/{drive-id}/root:/Account_ABC_12345
If:
200 → exists
404 → create folder
🔹 Step 5.6: Create Folder (If Needed)
Call:
Copy code

POST /drives/{drive-id}/root:/Accounts:/children
Body:
Json
Copy code
{
  "name": "Account_ABC_12345",
  "folder": {}
}
Now folder is created.
🔹 Step 5.7: Break Permission Inheritance
WHY? Because by default folder inherits parent permissions.
We want record-level security.
So break inheritance using SharePoint API.
Now folder has unique permissions.
🔹 Step 5.8: Remove Old Permissions
WHY? Because security may have changed.
Example: Old owner removed.
If you don’t remove old permissions → security mismatch.
So backend clears existing permissions.
🔹 Step 5.9: Apply New Permissions
For each user:
If Owner → write
If Shared → read
Call Graph:
Copy code

POST /invite
Body:
Json
Copy code
{
  "recipients": [{"email": "rahul@company.com"}],
  "roles": ["read"]
}
Graph updates SharePoint.
