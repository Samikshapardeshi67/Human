6. Technology Stack
The system is built using modern Microsoft cloud technologies:
Microsoft Dynamics 365 (Dataverse) – Record and security source.
Microsoft SharePoint Online – Document storage system.
Microsoft Graph API – Unified API to interact with SharePoint and Azure AD.
Dataverse Web API – Used to retrieve record ownership and sharing details.
Azure App Service – Cloud platform to host backend application.
Node.js / .NET Core – Backend development framework.
Azure Active Directory (Entra ID) – Identity and authentication provider.
OAuth 2.0 – Secure token-based authentication protocol.
7. Detailed Working Methodology
The system follows the below step-by-step process:
Step 1: Record Change Detection
When a record in Dynamics 365 is created, updated, reassigned, or shared, a trigger (Plugin or Power Automate flow) is activated.
Step 2: Backend Invocation
The trigger sends the Record ID and Entity Name to the Azure backend endpoint.
Step 3: Fetch Security Information
The backend calls the Dataverse Web API to retrieve:
Record Owner
Team Ownership (if applicable)
Shared Users
Access Rights (Read/Write)
Step 4: Identity Mapping
The backend retrieves user email addresses from Dataverse and calls Microsoft Graph API to obtain corresponding Azure AD Object IDs.
Step 5: Locate SharePoint Site and Library
The backend uses Graph API to:
Retrieve Site ID
Retrieve Document Library (Drive ID)
These identifiers are stored in configuration for reuse.
Step 6: Folder Creation or Retrieval
The backend checks if a corresponding folder exists in SharePoint. If not, it creates a new folder using a unique naming convention:
Copy code

EntityName_RecordName_RecordGUID
Step 7: Break Permission Inheritance
To ensure record-level security, the folder’s permission inheritance from parent library is disabled.
Step 8: Remove Existing Permissions
All previous custom permissions are removed to prevent mismatched access.
Step 9: Apply Updated Permissions
Based on Dataverse security:
Owner → Granted Edit/Write access
Shared users → Granted Read/Write access as applicable
Permissions are assigned using Microsoft Graph API.
Step 10: Logging and Error Handling
Each synchronization event is logged with:
Record ID
Timestamp
Status
Error details (if any)
8. Authentication and Security Model
The system uses Azure Active Directory (Entra ID) for secure authentication.
8.1 App Registration
An application is registered in Azure AD to obtain:
Client ID
Tenant ID
Client Secret
8.2 Access Token Generation
The backend uses OAuth 2.0 Client Credentials Flow to generate access tokens for:
Dataverse API
Microsoft Graph API
8.3 API Permissions
The following permissions are granted:
For Microsoft Graph:
Sites.ReadWrite.All
Files.ReadWrite.All
User.Read.All
For Dataverse:
user_impersonation
Admin consent is required to activate these permissions.
9. Expected Outcomes
The system is expected to:
Automatically synchronize security between Dynamics and SharePoint.
Eliminate manual permission management.
Reduce security inconsistencies.
Improve enterprise compliance.
Enable scalable and secure document access control.
