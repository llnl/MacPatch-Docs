### Table of Contents
* [Default Admin Account](#a1)
	* [Change Password](#a1a)
	* [Disable Admin Account](#a1b)
* [User Accounts](#a2)
	* [Standard Accounts](#a2a)
	* [Directory Accounts](#a2b)
	* [Azure Accounts](#a2c)
* [Account Rights](#a3)

#### Default Admin <a name='a1'></a>

The default admin account is created during the server setup ("ServerSetup.py --setup"). It is a local account stored in the MacPatch database with a hashed password, there is no default password and the account is no longer stored in the **siteconfig.json** file.

This account should only be used for setup of the MacPatch "Master" server. Once another admin account has been created, it's recommended that the default admin account be disabled.

Additional admin accounts can be created from the command line on the MacPatch server.

	cd /opt/MacPatch/Server/conf/scripts/setup
	sudo ./ServerSetup.py --config admin

##### - Change Default Admin password
<a name='a1a'></a>The default admin password is reset from the command line on the MacPatch server. The account is recreated with admin rights and a new password is generated and displayed. Please save it in a secure location, it can not be recovered.

	cd /opt/MacPatch/Server/apps/console/scripts
	sudo -u www-data /opt/MacPatch/Server/env/console/bin/python3 create_admin_account.py --username mpadmin --force

On Mac use "_appserver" instead of "www-data". To set a specific password instead of generating one add `--password "YourPassword"` (8 characters minimum).

##### - Disable Default Admin account
<a name='a1b'></a>To disable the default admin account, navigate to the Admin->Accounts menu, edit the account and set "Enabled" to "No".

**Upgrading from an older version:** If your **siteconfig.json** still contains a "users" -> "admin" account, it is no longer used and can be removed.

#### User Accounts <a name='a2'></a>

MacPatch supports 3 different account types not including the default admin account. MacPatch supports standard database accounts, LDAP/Active Directory accounts and Azure (Microsoft Entra ID) accounts.

<a name='a2a'></a>
##### Local Accounts
To add or remove a standard account, navigate to the Admin->Accounts menu. Simply click on the "+" icon just above the accounts table.

Assiging rights to a user account is fairly straight forward. If no rights are assigned the user can login and create a Client group and a patch group. This user will also be the owner of these groups and can assign additional users who can assist in managing these options.

<a name='a2b'></a>
##### Directory Accounts
MacPatch supports user accounts coming from LDAP/Active directory. During the server setup you were asked if you wanted to enable LDAP user support. If you answered "Yes", then your settings were saved to the "/opt/MacPatch/Server/apps/.mpglobal" file (the "LDAP_SRVC_" settings). To change the settings run:

	cd /opt/MacPatch/Server/conf/scripts/setup
	sudo ./ServerSetup.py --config ldap

Any user in the directory is allowed to login to MacPatch. This user will have the basic standard user rights.

Assiging rights to ldap users is just like a standard account. The only difference is the user has to login at least one time so that the user will show in the accounts table.

<a name='a2c'></a>
##### Azure Accounts
MacPatch supports user accounts from Azure (Microsoft Entra ID) using the Microsoft Authentication Library (MSAL). Users sign in with the "Login via Azure" button on the MacPatch console login page.

**Register the application in Azure**

1. In the Azure portal, go to "Microsoft Entra ID -> App registrations" and create a new registration.
2. Add a "Web" platform redirect URI: `https://<your-macpatch-console-host>/auth/signin-oidc`
3. Under "Certificates & secrets", create a new client secret. Save the secret value, it is only shown once.
4. Under "API permissions", make sure the Microsoft Graph "User.Read" (delegated) permission is granted.
5. Note the "Application (client) ID" and the "Directory (tenant) ID".

**Configure MacPatch**

Add the following settings to the "/opt/MacPatch/Server/apps/.mpconsole" file, then restart the console service.

	MSAL_CLIENT_ID=<Application (client) ID>
	MSAL_CLIENT_SECRET=<client secret value>
	MSAL_AUTHORITY=https://login.microsoftonline.com/<Directory (tenant) ID>
	MSAL_HTTP_SCHEME=https

	cd /opt/MacPatch/Server/conf/scripts/setup
	sudo ./ServerSetup.py --service MPConsole --action stop
	sudo ./ServerSetup.py --service MPConsole --action start

"MSAL_AUTHORITY" must be the full authority URL, not just the tenant ID. "MSAL_HTTP_SCHEME" should always be "https" in production.

**Login & Rights**

The MacPatch user name is the part of the Azure user name before the "@", e.g. "jdoe@example.com" will be "jdoe" in MacPatch. If a local or LDAP account with the same name exists, it is the same MacPatch account.

The account is created on the first login with no rights. Assiging rights to Azure users is just like a standard account, the user has to login at least one time so that the user will show in the accounts table. Azure app roles are not used, all rights are assigned in MacPatch.

To allow only Azure logins, add `LOCAL_AUTH_ALLOWED=no` to the "/opt/MacPatch/Server/apps/.mpconsole" file and restart the console service. This hides the "Login via MacPatch" button and disables local and LDAP logins, including the default admin account. Make sure at least one Azure account has been given admin rights first.

#### Account Rights <a name='a3'></a>

|Right |Description|
|---|---|
|Admin | This right gives the user admin rights through out the console.|
|AutoPKG | This right allows the user account to upload new packages using the autopkg tool.  |
|Agent Upload | This rtight allows the user to upload new MacPatch client agents. |
|API | This right allows the user to access MacPatch API's for automation. (Not Completed)|
