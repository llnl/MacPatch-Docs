## Table of Contents
- [Table of Contents](#table-of-contents)
	- [Prequisits \& Requirements](#prequisits--requirements)
		- [Requirements ](#requirements-)
		- [Perquisites ](#perquisites-)
		- [Linux Packages ](#linux-packages-)
	- [Download, Setup and Install ](#download-setup-and-install-)
		- [Get Software ](#get-software-)
		- [Install Software ](#install-software-)
		- [Setup Database ](#setup-database-)
		- [Configure Server Software ](#configure-server-software-)
		- [Database Setup \& Upgrades ](#database-setup--upgrades-)
		- [Start Configured Services](#start-configured-services)
		- [Stop Configured Services](#stop-configured-services)
	- [Server Setup \& Configuration ](#server-setup--configuration-)
		- [First Login ](#first-login-)
		- [Server Configuration ](#server-configuration-)
		- [Default Patch Group Configuration ](#default-patch-group-configuration-)
		- [Client Agent Configuration ](#client-agent-configuration-)
	- [Download \& Add Patch Content ](#download--add-patch-content-)

### Prequisits & Requirements
root or sudo access will be needed to perform these tasks.

#### Requirements <a name='a1'></a>
- Operating System:
	- macOS
		- Mac OS X 10.15 or higher
	- Linux
		- RHEL 8.x or higher
		- Ubuntu Server 18.04 (No longer being tested)
- Pyhton 3.12 or higher
- RAM: 4 Gig min
- MySQL 8.0.20 or higher

#### Perquisites <a name='a2'></a>
- MySQL installed (must have root password)
- If Installing on Mac OS X, **Xcode and command line developer tools** need to be installed **AND** the license agreement needs to have been accepted.
- Python 3.12 installed, if compiling please ensure libffi-devel is installed prior to compiling python.

##### Linux Packages <a name='a2a'></a>

The MacPatch server build script will attempt to install a number of required software packages there are a few packages that are recommended that be installed prior to running the build script.

**RedHat & CentOS**

RedHat & CentOS will require the "Development tools" group install. This group has a number of packages needed to build the MacPatch server.

	dnf groupinstall "Development tools"
	dnf install epel-release pcre-devel swig

**Ubuntu**

	apt-get install build-essential

### Download, Setup and Install <a name='a3'></a>

##### Get Software <a name='a3a'></a>
		mkdir /opt (If Needed)
		cd /opt
		git clone https://github.com/LLNL/MacPatch.git


##### Install Software <a name='a3c'></a>

		cd /opt/MacPatch/Scripts
		sudo ./MPBuildServer.sh

##### Setup Database <a name='a3b'></a>

The database setup script only creates the MacPatch database and the account needed to use the database. It also does some basic database configuration. Tuning the MySQL server is out of scope for this document.

The database script is a sql script. This file will need to be run by the MySQL database root account or an account that has the Grant option. 

Please edit the file, by setting the correct variable values for your MacPatch environment. On a standard build, only the default database account (mpdbadm) password needs to be set. Please remeber the mysql account name and password as it will be needed during the server setup.

It's strongly recommended that you delete the sql script once run. The file won't be needed again. Or, at the very least, remove the account password that was set. 

		cd /opt/MacPatch/Server/conf/scripts/setup
		MPDBSetup.sql (must be run on the MySQL server)

##### Configure Server Software <a name='a3d'></a>

		cd /opt/MacPatch/Server/conf/scripts/setup
		sudo ./ServerSetup.py --setup

The setup script will walk you through the following:

1. Server keys and default configuration.
2. Database settings, use the database account name and password from the database setup step.
3. LDAP/Active Directory login settings (optional).
4. Database schema & default data. The MacPatch database schema is created and the default data (patch group, software group, client group, agent configuration, etc.) is added.
5. First admin account. If no admin account exists you will be asked to create one. Enter an account name (default is "mpadmin") and a password, or leave the password blank to have a strong password generated for you.
6. Services to enable.

**Important:** The admin password is only displayed once, please save it in a secure location. It can not be recovered.

##### Database Setup & Upgrades <a name='a3e'></a>

If the database step was skipped during "--setup" or failed (e.g. the database was not reachable), it can be run on its own. This is also used to upgrade the database schema after updating the MacPatch software. It is safe to run more than once, existing data is not changed.

		cd /opt/MacPatch/Server/conf/scripts/setup
		sudo ./ServerSetup.py --setup-db

To create an additional console admin account from the command line

		sudo ./ServerSetup.py --config admin

**Note:** These commands run as the MacPatch service user (www-data on Linux, _appserver on Mac), there is no need to run them in the console virtual environment.

##### Start Configured Services

		cd /opt/MacPatch/Server/conf/scripts/setup
		sudo ./ServerSetup.py --service All --action start

##### Stop Configured Services

		cd /opt/MacPatch/Server/conf/scripts/setup
		sudo ./ServerSetup.py --service All --action stop

--

### Server Setup & Configuration <a name='a4'></a>

The MacPatch server software has now been installed and should be up and running. The server is almost ready for accepting clients. There are a few more server configuration settings which need to be configured.

#### First Login <a name='a4a'></a>
Login using the admin account and password created during “ServerSetup.py --setup”. You will need to login for the first time with this account to do all of the setup tasks.

There is no longer a default admin password, and the admin account is no longer stored in the **siteconfig.json** file. Admin accounts are stored in the MacPatch database with a hashed password.

Once these tasks are completed and other admin accounts have been created, it’s recommended that this account be disabled.

* Go to “Admin -> Accounts”
* Edit the account and set “Enabled” to “No”

If the admin password is lost, it can be reset from the command line. The account is recreated with admin rights and a new password is generated and displayed.

		cd /opt/MacPatch/Server/apps/console/scripts
		sudo -u www-data /opt/MacPatch/Server/env/console/bin/python3 create_admin_account.py --username mpadmin --force

On Mac use “_appserver” instead of “www-data”.

**Upgrading from an older version:** If your **siteconfig.json** still contains a “users” -> “admin” account, it is no longer used. Create a database admin account with “sudo ./ServerSetup.py --config admin” and then remove the “users” section from **siteconfig.json**.
#### Server Configuration <a name='a4b'></a>
Each MacPatch server needs to be added to the environment. The master server is always added automatically.

It is recommended that you login and verify the master server settings. It is common during install that the master server address will be added as localhost or 127.0.0.1. Please make sure that the correct hostname or IP address is set and that **"active"** is enabled.

* Go to “Admin -> Server -> MacPatch Servers”
* Double Click the row with your server or single click the row and click the “Pencil” button.

#### Default Patch Group Configuration <a name='a4c'></a>
A default patch group will be created during install. The name of the default patch group is “Default”. You may use it or create a new one.

To edit the contents for the patch group simply click the “Pencil” icon next to the group name. To add patches click the check boxes to add or subtract patches from the group. When done click the “Save” icon. (Important Step)

* Go to “Patches -> Patch Groups”
* Double Click the row with your server or single click the row and click the “Pencil” button.

#### Client Agent Configuration <a name='a4d'></a>

A default agent configuration is added during the install. Please verify the client agent configuration before the client agent is uploaded.

**Recommended**

* Go to “Admin -> Client Agents -> Configure”
* Set the following 3 properties to be enforced
	* MPServerAddress
	* MPServerPort
	* MPServerSSL
* Verify the “PatchGroup” setting. If you have changed it set it before you upload the client agent.
* Click the save button
* Click the icon in the “Default” column for the default configuration. (Important Step)
* Set MPServerAllowSelfSigned to 1 if your in a test environment and not using a valid SSL vertificate.

Only the default agent configuration will get added to the client agent upon upload.


--

### Download & Add Patch Content <a name='a5'></a>

**Apple Updates** <a name='a5a'></a>

Apple patch content will download eventually on it’s own cycle, but for the first time it’s recommended to download it manually.

The Apple Software Update content settings are stored in a json file (/opt/MacPatch/Server/etc/patchloader.json). By default, Apple patches for 10.9 through 10.12 will be processed and supported.

Run the following command via the Terminal on the Master MacPatch server.

**Linux**

	# sudo -u www-data /opt/MacPatch/Server/conf/scripts/MPSUSPatchSync.py

**Mac**

	# sudo -u _appserver /opt/MacPatch/Server/conf/scripts/MPSUSPatchSync.py

**Custom Updates** <a name='a5b'></a>

To create your own custom patch content please read the "Custom Patch Content" [docs](http://macpatch.llnl.gov/docs/4_custom-patch-content/).

To use "AutoPkg" to add patch content please read the "AutoPkg patch content" [docs](http://macpatch.llnl.gov/docs/7_packaging-autopkg/).
