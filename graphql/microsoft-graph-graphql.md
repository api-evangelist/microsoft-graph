# Microsoft Graph GraphQL Schema

## Overview

Microsoft Graph is the unified API gateway for Microsoft 365, Azure Active Directory (Entra ID), Teams, OneDrive, SharePoint, and security services. It exposes data and capabilities for over a billion users and thousands of enterprise organizations through a single, consistent REST endpoint at `https://graph.microsoft.com/v1.0`.

This GraphQL schema models the core resource types across the Microsoft Graph API surface, designed to help developers understand the shape of the data and relationships available through the API.

## Endpoint

- **REST Base URL:** `https://graph.microsoft.com/v1.0`
- **Documentation:** https://learn.microsoft.com/en-us/graph/api/overview
- **GitHub:** https://github.com/microsoftgraph
- **Graph Explorer:** https://developer.microsoft.com/en-us/graph/graph-explorer

## Authentication

Microsoft Graph uses OAuth 2.0 and OpenID Connect via Microsoft Entra ID (formerly Azure AD). Apps obtain tokens from:

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
```

Two permission types apply:

- **Delegated permissions** — The app acts on behalf of a signed-in user.
- **Application permissions** — The app acts as itself with no signed-in user (daemon/background service).

Scopes are declared per-resource (e.g., `User.Read`, `Mail.ReadWrite`, `Directory.ReadWrite.All`). The full permissions reference is at https://learn.microsoft.com/en-us/graph/permissions-reference.

## Schema Coverage

The GraphQL schema covers 70+ named types organized across the following domains:

### Identity & Users
- `User` — Core user profile, licenses, authentication, and activity
- `UserActivity` — Timeline activity entries for a user
- `AgreementAcceptance` — Records of Terms of Use responses
- `Authentication` — Container for a user's registered auth methods
- `AuthenticationMethod` (interface) — Base for all method types
- `PasswordAuth`, `PhoneAuth`, `EmailAuth`, `TOTPAuth`, `WindowsHello`, `FIDO2` — Specific auth method implementations
- `SignInActivity` — Last interactive and non-interactive sign-in timestamps

### Groups & Directory
- `Group` — Microsoft 365 and security groups
- `GroupLifecycle` — Expiration policy for groups
- `DynamicGroup` — Groups with dynamic membership rules
- `SecGroup` — Security-only groups
- `GroupSetting`, `SettingValue` — Group configuration
- `DirectoryObject` — Base type for all directory entities
- `Organization` — Tenant-level organization profile
- `VerifiedDomain`, `AssignedPlan` — Org metadata

### Applications & Service Principals
- `Application` — App registrations in Entra ID
- `ServicePrincipal` — Tenant-local app identity
- `AppRole`, `AppRoleAssignment` — Role definitions and assignments
- `PermissionGrant` — OAuth consent records
- `PermissionScope` — Published OAuth 2.0 delegated scopes
- `KeyCredential` — Certificate credentials on apps/service principals

### Conditional Access & Identity Protection
- `ConditionalAccessPolicy` — Sign-in policies with conditions and grant/session controls
- `NamedLocation` — IP/country locations for CA policies
- `RiskDetection` — Identity risk events from Entra ID Protection
- `SignIn` — Sign-in log entries with device, location, CA policy results

### OneDrive & Files
- `Drive` — Top-level document library (personal or SharePoint)
- `DriveItem` — File or folder within a drive
- `File`, `Folder`, `Package` — DriveItem facets
- `Permission`, `SharingLink` — Access control on items
- `Thumbnail`, `ThumbnailSet` — Preview images
- `IdentitySet`, `Identity` — Creator/modifier identity references

### Search
- `SearchRequest`, `SearchResult` — Search query and response
- `SearchHit`, `SearchHitsContainer` — Individual result hits
- `SearchEntityType` (enum) — Searchable content types (messages, files, sites, etc.)

### Teams & Communications
- `Team`, `Channel` — Teams and their channels
- `Chat`, `ChatMessage` — 1:1 and group chats with messages
- `TeamsApp`, `TeamsAppDefinition`, `TeamsAppInstallation` — Teams app catalog
- `TeamsTab` — Tabs within channels
- `ConversationMember` — Membership in teams/chats
- `OnlineMeeting` — Scheduled meetings
- `CallRecord`, `Session`, `Participant` — Call analytics
- `AudioConferencing`, `ChatInfo` — Meeting join details

### Scheduling (Shifts / Frontline Workers)
- `Schedule` — Team schedule container
- `Shift`, `ShiftItem`, `ShiftActivity` — Individual shift definitions
- `TimeOff`, `TimeOffItem`, `TimeOffRequest` — Time-off management
- `OpenShift` — Unassigned open shifts
- `TimeOffReason`, `SchedulingGroup` — Supporting schedule entities

### Devices & Intune
- `Device` — Azure AD registered/joined devices
- `DeviceManagement` — Intune management container
- `ManagedDevice` — Intune-enrolled devices with compliance details
- `CompliancePolicy`, `Configuration` — Intune policy types
- `PolicyAssignment` — Policy-to-group assignments

### Subscriptions & Change Notifications
- `Subscription` — Webhook registration for resource changes
- `ChangeNotification` — Notification payload delivered to the app
- `Delta` — Delta query response with next/delta links

### Licensing
- `SubscribedSku` — Tenant license subscriptions (seats, plans)
- `LicenseUnitsDetail`, `ServicePlanInfo` — SKU details

### Identity Governance
- `PrivilegedRole`, `PrivilegedRoleAssignment` — PIM role management
- `PrivilegedRoleSettings`, `PrivilegedRoleSummary` — PIM config and stats
- `AccessReview`, `AccessReviewDecision` — Periodic access reviews
- `AccessReviewReviewer`, `AccessReviewSettings` — Review configuration

### Audit & Security
- `Audit` — Directory audit log entries
- `SignIn` — Sign-in log entries
- `Alert` — Security alerts from Microsoft Defender / MSSP integrations
- `ThreatIntelligence`, `IntelProfile` — Threat actor profiles
- `IntelligenceProfileIndicator` — Threat indicators (IPs, domains, hashes)
- `HostSslCertificate` — SSL cert observations on hosts

### Bookings
- `BookingBusiness` — Microsoft Bookings business page
- `BookingAppointment` — Scheduled appointment
- `BookingService`, `BookingCustomer`, `BookingStaffMember` — Business configuration
- `BookingSchedulingPolicy`, `BookingWorkHours` — Scheduling rules

### Auth Artifacts
- `APIKey` — API key credential (preview)
- `Token` — OAuth 2.0 token response shape

## Key Relationships

```
User
 ├── Authentication → [PasswordAuth, PhoneAuth, EmailAuth, TOTPAuth, WindowsHello, FIDO2]
 ├── memberOf → [Group, DirectoryRole, AdministrativeUnit]
 ├── drive → Drive → [DriveItem]
 ├── activities → [UserActivity]
 └── agreementAcceptances → [AgreementAcceptance]

Group
 ├── members → [DirectoryObject]
 ├── owners → [DirectoryObject]
 ├── team → Team → [Channel, Schedule, TeamsApp]
 └── drive → Drive

Application
 ├── appRoles → [AppRole]
 ├── owners → [DirectoryObject]
 └── requiredResourceAccess → [RequiredResourceAccess]

ServicePrincipal
 ├── appRoleAssignments → [AppRoleAssignment]
 └── oauth2PermissionScopes → [PermissionScope]

Team
 ├── channels → [Channel] → messages → [ChatMessage]
 ├── members → [ConversationMember]
 ├── installedApps → [TeamsAppInstallation]
 └── schedule → Schedule → [Shift, TimeOff, OpenShift]
```

## OData Query Support

Microsoft Graph REST endpoints support OData query parameters that map naturally to GraphQL arguments:

| OData | GraphQL arg | Purpose |
|-------|-------------|---------|
| `$filter` | `filter` | Server-side filtering |
| `$search` | `search` | Full-text search |
| `$top` | `top` | Page size |
| `$skip` | `skip` | Pagination offset |
| `$select` | field selection | Return only specified fields |
| `$expand` | nested fields | Expand related resources |
| `$orderby` | (sort arg) | Sort results |

## Schema File

The full type definitions are in `microsoft-graph-schema.graphql` alongside this document.

## Related Resources

- API Reference: https://learn.microsoft.com/en-us/graph/api/overview
- Authentication: https://learn.microsoft.com/en-us/graph/auth/
- Permissions Reference: https://learn.microsoft.com/en-us/graph/permissions-reference
- SDKs: https://learn.microsoft.com/en-us/graph/sdks/sdks-overview
- Changelog: https://developer.microsoft.com/en-us/graph/changelog/
- Microsoft Graph GitHub: https://github.com/microsoftgraph
