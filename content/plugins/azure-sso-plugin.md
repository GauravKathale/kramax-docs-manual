---

title: 'Kramax Azure SSO Plugin'
weight: 75

menu:
  main:
    name: "Azure SSO Plugin"
    identifier: "user-guide-azure-sso-plugin"
    parent: "plugins"

---

## Integration Guide

**Plugin version:** `1.0.0`  
**Kramax:** `1.0.0`  
**Spring Boot:** `4.1.2`  
**Date:** `4 November 2026`

## 1. Executive Summary

Adding this plugin to a Kramax Spring Boot application replaces the Kramax username/password login with Microsoft sign-in (Azure Active Directory / Microsoft Entra ID). It needs one Maven dependency, an Azure app registration, and a set of properties. No application code is required.

### 1.1 What your application gets

- Single sign-on for the Kramax webapps (Monitoring, Tasklist, Admin) and the engine REST API.
- Azure-controlled access: Azure roles and groups become Kramax groups.
- Ready-made access levels: an administrator group and a read-only group, set up automatically at startup.
- Directory view: administrators see Azure groups and users in the Admin webapp.
- Secured REST API: `/engine-rest` requires sign-in and applies Kramax permissions.

### 1.2 What your team needs to do

| Step | Task | Owner |
| --- | --- | --- |
| 1 | Create and configure the Azure app registration | Azure administrator |
| 2 | Add the plugin dependency | Developer |
| 3 | Set the properties | Developer |
| 4 | Store the client secret safely | Developer / DevOps |
| 5 | Verify sign-in, access levels, and logout | Developer / QA |

## 2. How It Works

### 2.1 Sign-in sequence

1. A user opens a Kramax page without a session. The plugin sends the browser to Azure.
2. The user signs in at Azure. Azure returns the browser to `/login/oauth2/code/azure` with a one-time code.
3. The plugin exchanges the code for tokens and verifies the ID token with Azure public keys.
4. The plugin works out the user id and the user's Kramax groups from the token and Microsoft Graph.
5. The user lands on the post-login page. Kramax applies the permissions of those groups.

### 2.2 How a user gets a permission

Three separate things decide what a user can do. A user gets a permission only when all three are in place.

| Stage | Where | Question it answers |
| --- | --- | --- |
| 1. Membership | Azure | Which roles and groups does this user have? |
| 2. Acceptance | `roles-to-groups` or `user-group.allowed-group-ids` | Which of those does Kramax accept as Kramax groups? |
| 3. Permissions | Created by the plugin, or Admin > Authorizations | What may that Kramax group do? |

The plugin does not decide who may sign in. Anyone Azure allows can sign in, and a user with no accepted group sees almost nothing. To restrict sign-in itself, turn on **Assignment required** for the enterprise application in Azure.

## 3. Azure SSO Plugin

### 3.1 How sign-in works

{{< img src="../img/figure2-the-sign-in-sequence.png" title="The Sign-in Sequence" >}}

Kramax never sees the user's password. It receives signed tokens from Microsoft, checks them, and works out the user's Kramax groups. Roles and groups are read at sign-in, so a change in Azure takes effect the next time the user signs in.

### 3.2 Step 1: Register the application in Azure

1. In the Azure portal, open App registrations and create a new registration.
2. Add a redirect URI of platform type Web: `<public-url>/login/oauth2/code/azure`
3. Under Certificates & secrets, create a client secret and store its value safely.
4. Under App roles, create the roles `kramax-admin` and `kramax-readonly`.
5. Under Enterprise applications > Users and groups, assign those roles to users or groups.
6. Under API permissions, add the delegated Microsoft Graph permissions `GroupMember.Read.All` and `User.ReadBasic.All`, then grant admin consent.
7. Azure only issues the `email` claim for users with a mailbox. If some users have none, add `email` as an optional claim under Token configuration, or rely on fallback claims (see section 5.3).

Note: Using only app roles? You can leave out `GroupMember.Read.All` for sign-in. It is needed when access depends on Azure groups read from Microsoft Graph, and both permissions are needed to list users and groups in the Admin webapp.

### 3.3 Step 2: Add the dependency

```xml
<dependency>
  <groupId>org.vodafone.kramax.plugin</groupId>
  <artifactId>kramax-azure-sso-plugin</artifactId>
  <version>1.0.0</version>
</dependency>
```

### 3.4 Step 3: Set the properties

This is the recommended starting configuration. Replace values in `${...}` through environment variables or your secret store. Long lines may wrap on this page, but each property belongs on a single line.

```properties
# Turn on permission checks. Without this, every signed-in user can do everything.
kramax.bpm.authorization.enabled=true
# Needed for the history permissions of the read-only group
kramax.bpm.generic-properties.properties.enable-historic-instance-permissions=true

# Public URL of the application, as used in the browser
application.gateway-url=${GATEWAY_URL}

# Azure roles or groups (left) -> Kramax groups (right)
kramax.azure-activedirectory.roles-to-groups.kramax-admin=kramax-admin
kramax.azure-activedirectory.roles-to-groups.kramax-readonly=kramax-readonly
kramax.azure-activedirectory.groups-source=claim_and_graph
kramax.azure-activedirectory.redirection-base-uri=/login/oauth2/code/*
kramax.azure-activedirectory.logout-redirect-to-login=true
kramax.azure-activedirectory.logout-prompt=select_account

# Azure endpoints
spring.security.oauth2.client.provider.azure.user-name-attribute=email
spring.security.oauth2.client.provider.azure.authorization-uri=https://login.microsoftonline.com/${AZURE_TENANT_ID}/oauth2/v2.0/authorize
spring.security.oauth2.client.provider.azure.token-uri=https://login.microsoftonline.com/${AZURE_TENANT_ID}/oauth2/v2.0/token
spring.security.oauth2.client.provider.azure.jwk-set-uri=https://login.microsoftonline.com/${AZURE_TENANT_ID}/discovery/v2.0/keys

# Your app registration
spring.security.oauth2.client.registration.azure.client-id=${AZURE_CLIENT_ID}
spring.security.oauth2.client.registration.azure.client-secret=${AZURE_CLIENT_SECRET}
spring.security.oauth2.client.registration.azure.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.azure.scope=openid,profile,email,offline_access,https://graph.microsoft.com/GroupMember.Read.All,https://graph.microsoft.com/User.ReadBasic.All
spring.security.oauth2.client.registration.azure.redirect-uri=${application.gateway-url}/login/oauth2/code/{registrationId}
```

Important: Never commit the client secret or package it with the application. If it leaks, create a new one in Azure and remove the old one.

### 3.5 Step 4: Check sign-in

Start the application and open `<public-url>/kramax/`. You should be sent to Microsoft, then land on the Kramax dashboard. Section 3.7 has the full acceptance checks.

### 3.6 Access levels and permissions

{{< img src="../img/how-a-user-gets-a-permission.png" title="How a user gets a permission" >}}


A user only gets a Kramax group if the configuration accepts it. An Azure group that is not configured gives no access, so adding a group in Azure can never widen access in Kramax by accident.

| Access level | How to give it | What the user can do |
| --- | --- | --- |
| Administrator | Map a role or group to `kramax-admin` | Everything, on every resource |
| Read-only | Map a role or group to `kramax-readonly` | Open the webapps and view data, but not start or change anything |
| Custom | List the group in `user-group.allowed-group-ids`, then grant permissions in the Admin webapp | Only what you grant, and nothing until you do |
| No group | Nothing configured | Can sign in, but sees no data and has no webapp access |

Mapping an Azure group by object ID. Object IDs are safer than display names, because a group can be renamed:

```properties
kramax.azure-activedirectory.roles-to-groups.<azure-group-object-id>=kramax-readonly
```

Adding a custom group. The group ID in Kramax is the Azure object ID. Grant its permissions under Authorizations in the Admin webapp:

```properties
kramax.azure-activedirectory.user-group.allowed-group-ids[0]=<azure-group-object-id>
```

Default read-only permissions (groups get `ACCESS` to webapps and these permissions on all resource IDs):

| Resource | Permissions |
| --- | --- |
| `APPLICATION` | `ACCESS` on webapps in `read-only-applications` |
| `BATCH` | `READ`, `READ_HISTORY` |
| `DECISION_DEFINITION` | `READ`, `READ_HISTORY` |
| `DEPLOYMENT` | `READ` |
| `FILTER` | `ALL` |
| `GROUP` | `READ` |
| `HISTORIC_PROCESS_INSTANCE` | `ALL` |
| `HISTORIC_TASK` | `ALL` |
| `OPERATION_LOG_CATEGORY` | `ALL` |
| `PROCESS_DEFINITION` | `READ`, `READ_HISTORY`, `READ_TASK`, `READ_INSTANCE` |
| `PROCESS_INSTANCE` | `READ` |
| `TASK` | `READ` |
| `TENANT` | `READ` |
| `USER` | `READ` |

To change one, set it in the properties. Other resources keep defaults:

```properties
kramax.azure-activedirectory.read-only-permissions.TASK=READ,READ_VARIABLE
```

`FILTER ALL` allows read-only users to create, edit, and delete Tasklist filters, and `OPERATION_LOG_CATEGORY ALL` allows editing/deleting operation log entries. If this is not wanted, set:

```properties
kramax.azure-activedirectory.read-only-permissions.FILTER=READ
kramax.azure-activedirectory.read-only-permissions.OPERATION_LOG_CATEGORY=READ
```

Important: These grants are created only when the group has none yet for that resource. Changing the property later does not update an existing grant. Change it in the Admin webapp instead, or delete it there and restart so it is created again.

### 3.7 Sign-in and sign-out behaviour

Paths below use default webapp path `/kramax`.

| Request | What happens |
| --- | --- |
| Page load of `/kramax/app/**` without a session | Redirect to Microsoft sign-in |
| Script request to `/kramax/api/**` without a session | `401` |
| `/engine-rest/**` without a session | `401` |
| `POST` to `/engine-rest/**` without the CSRF token | `403`, and the session ends |
| Page load of `/kramax/login` | Azure sign-in, or post-login page if already signed in |
| Local admin setup pages | Blocked with `403`, because users come from Azure |
| Failed sign-in | Reason is logged and Microsoft account picker is shown. It needs a click, so it cannot loop |
| Sign-out | Kramax session ends and user returns to sign-in. Microsoft session stays active unless `rp-initiated-logout=true` |
| Expired access token | Renewed automatically with refresh token. If renewal fails, session ends |

## 5. Configuration Reference

Tables in 5.2 to 5.7 omit the prefix `kramax.azure-activedirectory.` for readability.

### 5.1 Azure connection (required)

Standard Spring Boot properties. Prefix: `spring.security.oauth2.client`. `azure` is the registration id.

| Property | Purpose | Value | If wrong |
| --- | --- | --- | --- |
| `provider.azure.authorization-uri` | Azure sign-in page | `https://login.microsoftonline.com/<tenant-id>/oauth2/v2.0/authorize` | Azure shows error or wrong tenant |
| `provider.azure.token-uri` | Exchange code for tokens | `.../<tenant-id>/oauth2/v2.0/token` | Sign-in fails after Azure page |
| `provider.azure.jwk-set-uri` | Public keys for token verification | `.../<tenant-id>/discovery/v2.0/keys` | Sign-in fails after Azure page |
| `provider.azure.user-name-attribute` | Token claim used as Kramax user id | `email` | Falls back to `user-name-fallback-claims` |
| `registration.azure.client-id` | App registration client id | `<client-id>` | Azure rejects sign-in |
| `registration.azure.client-secret` | App registration secret | `${AZURE_CLIENT_SECRET}` | Sign-in fails; `invalid_client` |
| `registration.azure.authorization-grant-type` | OAuth2 browser flow | `authorization_code` | Sign-in does not work |
| `registration.azure.scope` | Requested Azure permissions | See Step 3 | See scope table below |
| `registration.azure.redirect-uri` | Return address; must be in Azure app registration | `${application.gateway-url}/login/oauth2/code/{registrationId}` | Azure error `AADSTS50011` |

| Scope value | Why it is needed | If missing |
| --- | --- | --- |
| `openid,profile,email` | Sign-in and basic identity | Sign-in does not work properly |
| `offline_access` | Refresh token keeps session alive | Session ends when access token expires (often around one hour) |
| `GroupMember.Read.All` | Read groups and memberships from Microsoft Graph | Only token groups are used; warning logged |
| `User.ReadBasic.All` | Read names and emails of other users | Admins cannot list tenant users; group members are skipped |

`application.gateway-url` is your own property (public user-facing URL). It is used to build the redirect URI.

### 5.2 General

| Property | Default | Meaning and purpose | Not set / bad value |
| --- | --- | --- | --- |
| `enabled` | `true` | Switches plugin on/off without removing dependency. `false` restores username/password login. | Not set: plugin on |
| `registration-id` | `azure` | Client registration used by plugin. | No matching registration: startup fails with clear message |
| `identity-provider-enabled` | `true` | Provides signed-in user and groups to engine/webapps. | Not set: on |

### 5.3 Users and groups

| Property | Default | Meaning and purpose | Not set / bad value |
| --- | --- | --- | --- |
| `roles-to-groups.<key>` | None | Maps Azure app role/group name/group object id to Kramax group; case-insensitive. | Not set: no group-based access |
| `groups-source` | `claim` | `claim`, `graph`, or `claim_and_graph`; recommended `claim_and_graph`. | Bad value: startup fails |
| `user-group.allowed-group-ids` | None | Accept Azure group object ids as Kramax groups; id becomes Group ID. `all` accepts every group. | Unknown id ignored, warning logged |
| `user-group.allowed-group-names` | None | Same as above by display name; requires Graph. | Not set: no effect |
| `pass-through-unmapped-groups` | `false` | Accept every other Azure group as Kramax group too. | Not set: other groups ignored |
| `roles-claim` | `roles` | Token claim that holds app roles. | Wrong claim: roles not found |
| `groups-claim` | `groups` | Token claim that holds groups. | Wrong claim: token groups not found |
| `user-name-fallback-claims` | `preferred_username,upn,oid` | Claims tried in order when user-name-attribute missing. | None present: sign-in fails |

### 5.4 Permissions

| Property | Default | Meaning and purpose | Not set / bad value |
| --- | --- | --- | --- |
| `kramax.bpm.authorization.enabled` (full key) | `false` | Enables engine permission checks. | Not set: signed-in users can do everything |
| `kramax.bpm.generic-properties.properties.enable-historic-instance-permissions` (full key) | `false` | Enables history permissions for read-only group. | Not set: those history grants have no effect |
| `admin-groups` | `kramax-admin` | Groups given ALL on all resources at startup. | Not set: default applies |
| `read-only-groups` | `kramax-readonly` | Groups given default read-only permissions. | Not set: default applies |
| `read-only-applications` | `*` | Webapps read-only groups can open (`*` = all). | Not set: all webapps |
| `read-only-permissions.<RESOURCE>` | See section 3.6 | Overrides one resource permission set. | Bad resource/permission: startup fails |
| `create-group-authorizations` | `true` | Creates admin/read-only permissions at startup; existing entries are not changed. | `false`: manage all permissions manually |

### 5.5 Sign-in

| Property | Default | Meaning and purpose | Not set / bad value |
| --- | --- | --- | --- |
| `redirection-base-uri` | `/login/oauth2/code/*` | Callback path from Azure; must match `redirect-uri`. | Wrong path: `404` after Azure; blank: startup fails |
| `authorization-base-uri` | `/kramax/oauth2/authorization` | Path that starts sign-in; `/azure` is appended. | Not set: default applies |
| `login-page-redirect` | `true` | Redirect Kramax login page directly to Azure. | `false`: password form reachable but unusable for SSO users |
| `post-login-redirect-uri` | `/kramax/monitoring/dashboard` | Page opened after successful sign-in. | Not set: default applies |
| `post-login-failure-redirect-uri` | None | Page opened after failed sign-in. | Not set: Azure restarts sign-in with picker |
| `clear-session-storage-keys` | `kramax.sessionEnded` | Browser keys cleared after sign-in. | Empty may land users on login form again |

### 5.6 Logout

| Property | Default | Meaning and purpose | Not set / bad value |
| --- | --- | --- | --- |
| `logout-redirect-to-login` | `true` | After logout, go to Azure sign-in; if `false`, use `post-logout-redirect-uri`. | Bad value: startup fails |
| `logout-prompt` | None | Azure prompt after logout (`select_account` or `login`). Recommended `select_account`. | Not set/unknown: Azure session may auto-sign-in again |
| `rp-initiated-logout` | `false` | If `true`, also ends Azure session for that browser. | Bad value: startup fails |
| `end-session-uri` | Derived | Azure logout address for RP-initiated logout. | Not set: auto-derived |
| `post-logout-redirect-uri` | `/kramax/login` | Page after logout when `logout-redirect-to-login=false`. | Not set: default applies |

### 5.7 REST API and directory

| Property | Default | Meaning and purpose | Not set / bad value |
| --- | --- | --- | --- |
| `secure-rest-api` | `true` | Requires SSO session for `/engine-rest`, applies permissions, requires CSRF for writes. | `false`: API open without permission checks |
| `rest-api-path` | `spring.jersey.application-path`, else `/engine-rest` | REST API path to protect. | Wrong path leaves real API unprotected |
| `directory-enabled` | `true` | Shows Azure groups/users to administrators in Admin webapp. | `false`: everyone sees only themselves |
| `directory-scope` | `tenant` | `tenant` or `configured`; use `configured` for very large tenants. | Bad value: startup fails |
| `directory-cache-ttl` | `5m` | Cache duration for directory lookups. | Not set: default applies |
| `graph-base-uri` | `https://graph.microsoft.com/v1.0` | Microsoft Graph base URI. | Wrong URI: directory empty, warning logged |
| `graph-member-of-uri` | Graph `/me/transitiveMemberOf` query | Reads signed-in user's groups. | Wrong query: Graph groups missing, warning logged |
| `graph-connect-timeout` | `5s` | Connect timeout to Graph. | Too low: lookups fail |
| `graph-read-timeout` | `10s` | Read timeout for Graph responses. | Too low: lookups fail |

### 5.8 Production server settings

Standard Spring Boot properties.

| Property | Value | Purpose | Needed? |
| --- | --- | --- | --- |
| `server.forward-headers-strategy` | `native` | Trust gateway `X-Forwarded-*` headers for public-host/https redirects. | Yes, behind a gateway |
| `server.servlet.session.cookie.same-site` | `lax` | Sends cookie on Azure return but blocks cross-site form posts. | Yes |
| `server.servlet.session.cookie.secure` | `true` | Sends session cookie over https only. | Yes on https |
| `server.servlet.session.timeout` | `30m` | Idle session lifetime. | Optional |
| `server.servlet.session.cookie.http-only` | `true` | Prevents scripts from reading session cookie. | Default already true |

## 6. Production Checklist

- [ ] Client secret comes from a secret store and has owner + expiry reminder.
- [ ] Application runs on https with `server.servlet.session.cookie.secure=true`.
- [ ] Forwarded headers work: redirects use public host + https.
- [ ] Redirect URI in Azure exactly matches public URL.
- [ ] `kramax.bpm.authorization.enabled=true` is set.
- [ ] `secure-rest-api` remains `true`.
- [ ] Admin consent is granted for `GroupMember.Read.All` and `User.ReadBasic.All`.
- [ ] Persistent database is configured.
- [ ] `directory-scope` suits tenant size.
- [ ] Read-only permissions and app access are reviewed.
- [ ] All verification checks pass with real Azure accounts.

## 7. Troubleshooting

Start with application logs. Look for:

- `SSO login failed [code]: reason`
- `SSO login of ... with Kramax groups [...]`

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Application does not start | Invalid property value, missing `${...}`, or no client registration | Check startup error and section 5 |
| Azure shows `AADSTS50011` | Redirect URI not registered | Register exact `redirect-uri` in Azure |
| `404` after Azure sign-in | `redirection-base-uri` and `redirect-uri` path mismatch | Remove property or correct the path |
| Account picker keeps returning after sign-in | Sign-in is failing | Check `SSO login failed [code]`; `invalid_client` usually means wrong/expired secret |
| User signs in but sees nothing | No accepted group | Check groups in logs, Azure assignment, and `roles-to-groups` |
| Group name mapping gives no access | `groups-source` not using Graph | Set `groups-source=claim_and_graph` |
| `Could not read groups from Microsoft Graph` warning | Missing permission or no admin consent | Grant permission and include it in scope |
| Admin sees only themselves in Users page | `User.ReadBasic.All` missing or user not admin | Grant permission, add to scope, sign in again |
| `does not exist in this tenant` warning | Object id in `allowed-group-ids` is wrong or from another tenant | Copy id from Azure group |
| Group is listed but gives no rights | Group is listed for display only | Add it to `allowed-group-ids` or `roles-to-groups` |
| Header shows email instead of full name | Missing token name claims | Add `given_name` and `family_name` optional claims |
| User is signed in again right after logout | `logout-prompt` not set | Set `logout-prompt=select_account` |
| Redirects go to internal host or http | Forwarded headers not trusted | Set `server.forward-headers-strategy`; check proxy settings |
| Scripts get `401` from `/engine-rest` | Browser SSO session required | Use API access patterns in section 8 |
| Azure changes do not appear | Directory cache or stale user session | Wait for `directory-cache-ttl`; sign in again |

## 8. API Clients and Automation Notes

The plugin secures `/engine-rest` for browser SSO sessions. Scripted clients and machine-to-machine flows are not covered by this browser login flow.

- Use dedicated API authentication architecture for automation.
- Keep `secure-rest-api=true` in production.
- Do not relax API security to support scripts unless you implement an approved alternative.


