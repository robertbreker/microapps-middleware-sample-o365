

# Citrix Workspace Microapps Middleware for O365

microapps-middleware-sample-o365 is sample code that shows how Microsoft
Office 365 can be integrated into Citrix Workspace using Microapps.

The implemented features are:
 - Near real-time experience powered by Microsoft Graph's event subscription
   and Microapps' webhooks.
 - Multi-tenant middleware service, built using Flask, that scales to being used by
   many Workspace and O365 environments
 - Comes with ready-to-import and -use Microapp integration bundle.
 - Providing Calendar functionalities:
  - Calendar and event view over the next seven days.
  - Notifications for upcoming meetings.
  - Join virtual meetings directly from within Workspace.
  - Notifications for new meeting requests.
  - Respond to meetings Accept, Tentative, Decline with optional text.
  - Delete meeting events without responding.
  - Direct access to events in web-based Outlook.
 - Providing Email functionalities:
  - Notifications about unread high-priority emails.
  - Notifications about unread emails from a Workspace user's manager.
  - Direct access to email in web-based Outlook
  - Send Emails

## Getting Started

### Prerequisites
- Citrix Workspace with Microapp entitlement - [refer to this getting started guide](https://developer.cloud.com/citrix-workspace/build-workspace-microapp-integrations/docs/getting-started) for setting up a free developer instance
- Office 365 - note that the [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program] comes with a free developer test instance
- Admin access to AAD (comes with Office 365
- A publicly routed host to run the Flask middleware (any public cloud will work nicely)


### Get and deploy the multi-tenant service - 
Get the middleware.
```
git clone https://github.com/robertbreker/microapps-middleware-sample-o365.git
```

Deploy the middleware.
```
cd microapps-middleware-sample-o365.git
Todo...
```
Note that this middleware service can be used by multiple Citrix Workspace tenants and configured integrations.

### Deploy the integration to your Citrix Workspace Microapp environment
For each tenant (combination of O365 and Citrix Workspace deployment) -

Configure an app client and secret for Microsoft Graph using the Azure Portal.

1. Head to [App registrations](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps]) for your Azure Active Directory.
2. Create a "New registration".
3. Enter a user facing name, such as "microapps-middleware-sample-o365".
4. Complete the operation by selecting "Register"
5. Take note of the "Application (client) ID", that is the client id needed for Workspace later.
6. Select Certificates & secrets from the left menu
7. Select New client secret
8. Provide a descriptive name, like "workspace-integration" and click Add
9. Take note of the "Value", i.e. the secret, that is the Client secret needed for Workspace later.

Configure the API permissions for the new app client. 
1. Select API permissions in the left menu.
2. Select Add a Permission -> Microsoft Graph -> Application Permission
    1. Tick Calendars read
    2. Tick Mail.read
    3. Tick User.Read.All
3. Select the following delegated permissions
    1. Tick Calendars.ReadWrite
    2. Tick Mail.Send
4. Select Add permissions to complete the operations
 
 Configure the app client authentication.
1. Select "Authentication from the left menu"
2. Select "Add a platform", "Web" and add two redirect URIs
    1. https://\<yourcustomerid\>.\<yourgeo\>.iws.cloud.com/app/api/auth/serviceAction/callback
    2. https://\<yourcustomerid\>.\<yourgeo\>.iws.cloud.com/admin/api/gwsc/auth/serverContext

Import and configure the integration bundle.

1. Login to (Citrix Cloud)[https://cloud.citrix.com/].
2. Select "Manage" for the Microapps tile
3. Select "Manage", "Add Integration", "Import a previously configured integration".
4. Select SampleO365Middleware.service.mapp and "Import"
5. Right click the Integration, select "Edit", in the left menu "Configuration".
6. Configure the previously noted "Client ID", and "Client Secret" and select "Login with Your service account". Follow the dialog for providing access
7. Also configure the "Client ID" and "Client Secret" for Service Action Authentication.
8. Store the settings with Save
9. Select "Webhook Listeners". Take note of the URLs of email\_webhook and calendar\_webhook, remembering which is which.
10. Select "Data loading", edit the "trigger\_middleware" data endpoint.
11. Edit the query paramters to contain the calendar\_webhook and email\_webhook URLs, that you just noted.
12. Subscribe users and try the integration.

## Design notes

The middleware \(app.py\), serves to implement the glue between Microapps and
Microsoft Graph. It registers for event callbacks from Graph, processes the
callbacks, and forwards processed data to Microapps. The integration bundle
\(SampleO365Middleware.service.mapp\) implements a Microapp integration on top
of the middleware.

Here's how it all works in a nutshell:
  - Microapps implements the SOR authentication
  - The middleware is regularly called by a data-endpoint in Microapps, with a
    valid bearer token for the SoR
  - The middleware caches the bearer token.
  - The middleware subscribes to SoR events and renews subscriptions every 24h.
  - Notably the subscription is done with a separate API call for each user.
  - When the SoR calls back with events, the middleware makes subsequent calls
    to the SoR for obtaining the actual data of interest.
  - Service actions are passed-through to the SoR by the middleware.

