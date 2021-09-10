<!--Linux Install Training - Skytap Environment-->
Author: Phil D'Amico [pdamico@tableau.com](mailto:pdamico@tableau.com)

# Summary

This training provides a sandbox for practicing an install of Tableau Server on Linux. This training is delivered in Skytap, with 3 Linux VMs.

You will install Tableau Server on Linux, configure LDAP as the Identity Store, and implement a basic Content Model, using three LDAP Groups.

Here's a summary of the training:

* Install Tableau Server

* Configure LDAP as an Identity Store

* "Lock" the Default Project (Best Practice)

* Import LDAP Groups through the Web UI **and** command line  

* Install Drivers to connect to SQL Server from Linux  

* Add and configure a 2nd Node

When finished, you will have:

* A running, 2-node Tableau Server cluster with LDAP Identity Store / Local Authentication

* Access to SQL Server: **Superstore World** and **DEV Superstore World**

* Practice reading in groups from LDAP

* Practice with the Linux Command Line

* Basic understanding of Content Model Best Practices

* A working LDAP Server you can browse

For a comprehensive guide to installing Tableau Server on Linux, refer to [**Install and Configure Tableau Server**](https://help.tableau.com/current/server-linux/en-us/install_config_top.htm).  

# Your Environment (3 Machines)  

- Tableau Server node1 (Hostname: **node-1**)  

    * XUbuntu Desktop 18.04 LTS, 32GB RAM, 8 Cores, 80GB Disk
    * user/password: **node1 / node1**
    * Tableau Server install (.deb) file on Desktop
    * **register.json**: JSON file for Registering Tableau Server via command line
    * **config.ldap.json**: JSON file to configure LDAP Identity Store (see below)
    * Apache Directory Studio (LDAP Browser) configured to connect to LDAP Server (see below)  

- Tableau Server node2 (Hostname: **node-2**)

    * XUbuntu Desktop 18.04 LTS, 32GB RAM, 8 Cores, 80GB Disk
    * user/password: **node2 / node2**
    * Tableau Server install (.deb) file on Desktop

- LDAP Server, SQL Server  (Hostname: **train-vm**)  

    * Ubuntu Desktop 20.04 LTS, 8GB RAM, 4 Cores, 50GB Disk
    * user/password: **train / train**
    * LDAP Server (root: "dc=training,dc=com")
    * SQL Server
    * Apache Directory Studio (LDAP Browser)

# Launch Environment

* Press Power (Play) button above the machines  

* Click on **node1** to launch in your browser  

* **Wait**. The VM may not be responsive at first; it can takes up to 2 minutes to load the VM.    

* **Review Toolbar at top of screen**.  *If you don't see the dock at the bottom of the screen, click the **Auto Fit** icon in the Skytap toolbar.*

* Open a Terminal window. There are two ways to do this:  

    - Click the **Terminal** icon on the dock at the bottom of the screen  
    - Right-click the Desktop. Click **Open Terminal Here**  

* Your prompt should look like this:  

    `node1@node-1:~/Desktop$`  

* If it does not, enter the following at the prompt:  

    `cd /home/node1/Desktop`


# Prepare the Environment

## Update Linux Repositories

**apt-get** is the **Package Manager** for this Linux distribution. The next 3 commands will:

* Update the repositories on the machines. This ensures you have access to the latest revision of this distribution  

* Upgrade all packages to the latest version  

* Install the package you need to install **.deb** files, which is how Tableau Server is distributed  


**sudo** is the command that allows you to run "as root". Enter the following commands. Accept all prompts. **Note:** The "sudo" password is **node1**

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get -y install gdebi-core
```

## Install and Initialize TSM  

* Confirm that you are at the Desktop. Your prompt should look like this:  

    `node1@node-1:~/Desktop$`

* If it doesn't look like this, change to the Desktop directory:  

    `cd /home/node1/Desktop`

* Install Tableau Server package (1:30). Enter the following in a Terminal window.  **Note** the ".deb" file may be different than what is listed here. Adjust accordingly.

    `sudo gdebi -n tableau-server-2021-3-0_amd64.deb`  

* Initialize TSM (1:00)  **Note** the file location may be different than what is listed here. Adjust accordingly. The command is displayed at the end of the previous command.

    `sudo /opt/tableau/tableau_server/packages/scripts.20213.21.0902.2150/initialize-tsm --accepteula`  

* **Exit Terminal**. Type "exit" and press Enter, or press **Ctrl**-D  

## Logout, then Login  


* **Logout completely**. Click the menu in the upper-left of the screen, or the "whisker" menu at the bottom. In either case, when you see the menu, locate and click the **Power** icon, then click **Log Out**  

* **Login** (password: **node1**)  

## Activate and Register Tableau Server  

- Open Terminal Window  

- Confirm that you are at the Desktop. Your prompt should look like this:  

    `node1@node-1:~/Desktop$`

- If it doesn't look like this, change to the Desktop directory:  

    `cd /home/node1/Desktop`

- Activate Tableau Server with the license key supplied by your instructor  

    `tsm licenses activate -k <your_license_key>`  

- **Optional**: review the registration JSON file. You use this file instead of entering the commands in a Web GUI. Remember: the goal of this tutorial is to perform an install **entirely** from the command line. Enter the following at the prompt (**cat** is the Linux command equivalent to **type** on Windows):  

    `cat register.json`

- Register  

    `tsm register --file register.json`  

# Configure LDAP

## Optional: Review LDAP Server Presentation

**Ctrl**-Click or **Cmd**-Click links to open the presentation in a new window.

Link to Google Slides: [LDAP and Tableau Server: Shared](https://docs.google.com/presentation/d/1cx-90_mRHFWr5HVBXybnoWecAP21MHciMIdf80EYYp8/edit?usp=sharing).  

## Test LDAP Connection

Tableau Server has to be able to connect to your LDAP Server. One way to validate this connection is to use a GNU/Linux utility `ldapsearch`.  

Enter the following in a Terminal window:

```
ldapsearch -h train-vm \
-D "uid=admin,o=Users,dc=training,dc=com" \
-w admin -b "dc=training,dc=com"  
```

**Note**: Be sure to enter the backslash at the end of each line, then press Enter. This technique lets you enter a command on multiple lines.


| Parameter | Description |
| :-----: | -------- |
|   -h     | Host  |
|   -D     | Distinguished Name  |
|   -w     | Password for the Distinguished Name (admin user, in this case) |
|   -b     | Base (where to start searching; in this case at the ROOT entry) |  


This will list all the entries in the LDAP directory. You can scroll back through to review. Also, you can repeat the command and "pipe" the output. Press the **Up-Arrow** key to repeat the command, this time piping the output to **more**:  

```
ldapsearch -h train-vm \
-D "uid=admin,o=Users,dc=training,dc=com" \
-w admin -b "dc=training,dc=com"  | more
```

Press **\<space\>** to scroll. Press **q** to exit the command and return to a prompt. If you reviewed the LDAP Presentation, this should look reasonably familiar. The LDAP Server has been pre-populated with:  

* admin user (password: admin)
* Three groups: Sales, Dev, HR
* 5 users in each group: Sales01-Sales05, HR01-HR05, Finance01-Finance05
* All passwords are **tableau** (except for admin)

## Configure LDAP Identity Store

* Import settings

    `tsm settings import -f /home/node1/Desktop/config.ldap.json `

* Test the configuration: find a User  

     `tsm user-identity-store verify-user-mappings -v dev01`

* You should see this:

```
      Distinguished Name: uid=Dev01,o=Users,dc=training,dc=com
                    GUID: Not found
                Username: Dev01
            Display Name: Dev01
                   Email: Dev01@training.com
              JPEG Photo: Not found
               Thumbnail: Not found
```

* Test the configuration: find a Group

     `tsm user-identity-store verify-group-mappings -v Dev`

* You should see this:

```
      Distinguished Name: cn=Dev,o=Groups,dc=training,dc=com
                    GUID: Not found
             Description: Development
                   Email: Not found
       Number of members: 5
```

## If you have errors...

Once you initialize Tableau Server (`tsm initialize` command below), you can’t use this JSON import method (`tsm import -f config.ldap.json` in our example) to make changes. You have to use these commands to make individual changes. Alternatively, you can use the tsm user-identity-store commands.

Here is a list of common configuration commands if you want to adjust your LDAP Identity Store values after you install Tableau Server.  

The format is `tsm configuration set -k <key> -v <value>`  

Example: `tsm configuration set -k wgserver.domain.fqdn -v “train-vm”`

```
tsm configuration set –k wgserver.domain.ldap.bind –v simple
tsm configuration set -k wgserver.domain.ldap.group.baseDn -v "o=Groups,dc=training,dc=com"
tsm configuration set -k wgserver.domain.ldap.group.baseFilter -v "objectClass=groupofNames"
tsm configuration set -k wgserver.domain.ldap.hostname -v “training.us”
tsm configuration set -k wgserver.domain.ldap.user.baseDn -v  "o=Users,dc=training,dc=com"
tsm configuration set -k wgserver.domain.ldap.user.baseFilter -v "objectClass=inetOrgPerson"
tsm configuration set -k wgserver.domain.password -v “admin”
tsm configuration set -k wgserver.domain.username -v “cn=admin,dc=training,dc=com”
```

Refer to [**tsm user-identity-store**](https://help.tableau.com/current/server/en-us/cli_user-identity_tsm.htm) for a complete list  

# Finish Installation

## Initialize Tableau Server (15:00)  

```
      tsm pending-changes apply  

      tsm initialize --start-server --request-timeout 1800  
```
***Take a break. This should take at least 12-15 minutes.***  

## Enable Metadata Services

```
      tsm maintenance metadata-services enable
```
## Add Administrator Account  

```
      tabcmd initialuser \
      --username 'admin' \
      --password 'admin' \
      --server http://localhost
```

## Initial login  

* Launch Web Browser (Chromium)  
* Navigate to <http://localhost>  
* User **admin** Password: **admin**  


# Congratulations!

You have installed Tableau Server on Linux. In the next section, you will:  

- "Lock" the Default Project (Best Practice)  
- Import 2 groups of users, using the UI **and** the command line  
- Assign permissions to Groups (Best Practice)  
- Configure Tableau Server to connect to SQL Server  
- Test Connectivity to SQL Server by creating a Workbook  

# Content Model: Do this First

* Login as Server Administrator (admin / admin)  

* Modify Permissions for Default Project. Remove ALL permissions from the **All Users** Group  

  1. Click **Explore**  

  1. Click **...** next to **Default** Project  

  1. Click **Permissions..**  

  1. Note permissions for **All Users**.  

  1. For each tab, set Permissions to **None**  (click drop-down under **Template**)  

  1. Make sure you have set it for each content type: Projects, Workbooks, Data Sources, Data Roles, Flows, Metrics  

  1. Click **Save**  

  1. Click **X** in upper-right to close the Window  


 **Why should I remove all Permissions?** When you create a Project, Tableau Server uses the **Default** Project as the template. If you deny all permissions to the **All Users** Group in the **Default** Project, you are guaranteed that any new Project will be locked until you explicitly grant permissions on it.

 **What happens if I don't?** When you create and save content, Tableau Server sets the permissions on that content based on the Permission structure of the Project. If you subsequently change the Permission structure, those changes **do not** propagate to content that is already in the Project. This could compromise the security of the content.

# Groups and Permissions

Reference: [**Tableau Server Help: Permissions**](https://help.tableau.com/current/server-linux/en-us/permissions.htm#evaluate-permissions-set-at-multiple-levels)  

## Import Finance Group: Web UI

* Login as Server Administrator (admin / admin)  

* Create Finance Project

    * Click **Explore**

    * Click **New** -> **Project**

    * Enter **Finance**

    * Click **Create**  

* **Optional:** Review Permissions on Finance Project  

    * Click **...** next to Finance Project

    * Click **Permissions..**

    * Note permissions for **All Users**. They should be **None**

    * Click **X** in upper-right to close the Window  

* Import Finance Group  

    * Login as Server Administrator (admin / admin)  

    * Click **Groups**

    * Click **Add Group**; Select **Active Directory Group**

    * In the search bar, enter **Finance**. Select it

    * Set all to **Creator** (drop-down list)

    * Click **Import**

    * Wait a few seconds. The **Finance** Group should appear

    * Click **Finance** Group

    * Set **Minimum Site Role** to **Creator**  

    * Click **Import**. Shortly, you should see a message at the top of the screen

    ```
        Info:Completed adding group “Finance” on domain “train-vm”.
        5 users were added and 0 users were removed from the group
        5 users were added to the site. 5 users site roles were updated.
    ```

    * To confirm, click **Users**. You should see **Finance01-Finance05**

    * To confirm, click **Groups**. You should see **All Users** and **Finance**


## Set Permissions for Finance Group  

* **Optional**: Check Permissions for Finance01 User  

    * Sign Out

    * Sign in **User**: Finance01 **Password**: tableau

    * Click **Explore**  

    * Can you see the **Finance** Project? Why not?. You should not see the **Default** Project either  

* Modify Permissions on Finance Project  

    * Sign out of **Finance01**  

    * Sign in as Administrator (admin / admin)  

    * Click **Explore**  

    * Click the three dots **...** for the **Finance** Project  

    * Click **Permissions**  

    * Click **+ Add Group/User Role** and Select the Finance **Group** (not users)  

    * For each group of Permissions, Select **Publish** from the **Template** drop-down  

    * Click **Save**. Note the updated **Effective Permissions** at the bottom  

    * Click **X** in upper-right to close Window  

* **Optional**: Check Permissions for Finance01 User  

    * Sign Out

    * Sign in **User**: Finance01 **Password**: tableau

    * Can you see the **Finance** Project?

    * Note that you *can't* see the **Default** Project. Why?

## Import Sales Group: Command Line

* Right-click the Desktop. Click **Open Terminal Here**  

* **Optional** Review the tabcmd login command:  

    `tabcmd help login`  

* Login to the server via tabcmd  

   `tabcmd login --username admin --password admin --server localhost`  

    You should get something like this:
```
    ===== Creating new session
    =====     Server:   http://localhost
    =====     Username: admin
    ===== Connecting to the server...
    ===== Signing in...
    ===== Succeeded
```  
* **Optional**: Review the tabcmd syncgroup command: `tabcmd help syncgroup`

* Enter the following command:
            `tabcmd syncgroup Sales -r Creator`  

    You should get something like this:
```
    ===== Continuing previous session
    =====     Server:   http://localhost
    =====     Username: admin
    ===== Synchronizing server with Active Directory group Sales...
    ===== 0% complete
    ===== 100% complete
    ===== Summary of import of group 'train-vm\Sales':
    Users added to group: 5
    Users added to site: 5
    Users with insufficient licenses: 0
    Users in Active Directory group: 5
    Users processed: 5
    Users skipped: 0
    Users with information updated: 0
    Users with site role updated: 5
    Users removed from group: 0
    Users unlicensed: 0
```

* **Optional**: Confirm the Sales Group via Web UI

    * Login as Server Administrator (admin / admin)  

    * Click **Groups**. Do you see the Sales Group?

    * Click **Users**. Do you see the Sales01-Sales05 Users?

## Optional: Create Sales Project, Assign Permissions

* Create Sales Project  

    * Click **Explore**

    * Click **New** -> **Project**

    * Enter **Sales**

    * Click **Create**

* Modify Permissions on Sales Project  

    * Click **Explore**

    * Click on three dots "..." for the **Sales** Project

    * Select **Permissions**

    * Click **+ Add Group/User Role** and Select the Sales **Group** (not users)

    * For each group of Permissions, Select **Publish** from the **Template** drop-down

    * Click **Save**. Note the updated **Effective Permissions** at the bottom  

    * Click **X** in upper-right to close Window  

* Confirm Permissions  

    * Sign Out

    * Sign in **User**: Sales01 **Password**: tableau

    * Click **Explore**

    * Can you see the **Sales** Project?

    * Can you see the **Finance** Project?

# Install SQL Server Driver (Linux)  

SQL Server is installed on **train-vm**. You have to install the SQL Server driver on the Tableau Server.  

## Install UnixOdbc Driver

  At a terminal, enter the following (**node1** is the sudo password):  

  `sudo apt install unixodbc`

## Install Tableau SQL Server Driver

* Launch Web Browser (Chromium)  

* Navigate to [**Driver Download**](https://www.tableau.com/support/drivers). Alternatively, google **tableau driver download** and click on the first link (**Driver Download - Tableau**)  

* Scroll to **Microsoft SQL Server**  

* Follow instructions under **On Debian and Ubuntu Linux distributions:**  

* To install the driver, run the following command:  

  `sudo dpkg -i /home/node1/Downloads/msodbcsql17_17.5.1.1-1_amd64.deb`  

* Accept the License terms (Press **\<Tab\>** key to highlight **Yes**; press **Enter**)  

# LDAP Browser

## Load Apache Directory Studio

- On the Desktop, Double-click **LDAP Browser** icon  

- If you don't see a **Connection** window with a **Training** entry, click **Window** -> **Open Perspective** => **Other** -> **LDAP**  

- Double-click on **Training** in the **Connection** window (lower-left)

## Review LDAP Structure

- Expand Users and Groups  

- Note the attributes unique to each entry  

    - `inetOrgPerson` for Users  

    - `groupOfNames` for Groups  

# Create a Workbook

* Login Sales01 / tableau

* Click **Explore**

* Click **New** -> **Workbook**

* Click **Connectors**

* Click **Microsoft SQL Server**

* Connection

    - Server: **train-vm**
    - Database: **Superstore World**
    - Authentication: **Use a specific username and password**
    - User: **test**
    - Password: **T@bleau2021!**

<!--Add a node2-->

# Add a Node  

To add a node, you need information about the "initial node". The next command - run from the terminal on the "initial node" - stores this information in a JSON file. You then copy that file to the new node and use it to initialize that node.

## Create the Bootstrap File  

```
tsm topology nodes get-bootstrap-file --file /home/node1/Desktop/bootstrap.json  
```

## Copy Bootstrap file to node-2

```
rsync /home/node1/Desktop/bootstrap.json node2@node-2:.  
```
## Login to node-2

```
ssh node2@node-2  
```

## Install Tableau Server on node-2

The following commands are identical to the ones you executed to install Tableau Server EXCEPT the last command. The last command takes your JSON file as a parameter so it can be joined to the cluster.  

```
sudo apt update
sudo apt upgrade  
sudo apt -y install gdebi-core  
sudo gdebi -n /home/node2/Desktop/tableau-server-2021-3-0_amd64.deb
cd /opt/tableau/tableau_server/packages/scr (press <tab> to auto-complete)
sudo ./initialize-tsm -b /home/node2/bootstrap.json --accepteula

```
## Logout from node-2  

Type "exit" and press Enter, or press **Ctrl**-D. You should still be at a terminal, but now you are back in node-1

## Set Topology from terminal on node-1  

```
tsm topology set-process -n node2 -pr clustercontroller -c 1
tsm topology set-process -n node2 -pr backgrounder -c 4
tsm topology set-process -n node2 -pr gateway -c 1
```

## Apply Changes (10:00)  
```
tsm pending-changes apply  
```
## Verify

* Launch Web Browser (Chromium)  
* Navigate to <https://localhost:8850>  
* User **node1** Password: **node1**  

# Summary: Lessons Learned

What have you done:

- You performed a complete install of a multi-node Tableau Server on Linux entirely from the command line.

- You practiced implementing security on projects using Groups


What you now know:  

- You know what a Linux distribution is, why there are so many of them, and that it's NOT just Linux (remember, "Linux" is just the kernel). **Side Note**: Purists do not call it "Linux", but "GNU/Linux".

- You know the difference between an "Identity Store" and "Authentication"

- You learned some techniques to make you more productive at the Linux command line

	- **History is your friend**. Use the up-arrow to scroll back. Note also that history (the last 1,000 commands) is preserved from one session to the next (unlike Windows). Command history is stored a regular text file in your home directory, which you can query using grep (`~/.bash_history`)  

	- With regards to `~/.bash_history` above, you now know that the `~` sign is shorthand for the user's home directory. You also know that a filename that begins with `.` is a hidden file. It won't show in directory listings (`ls`) unless you ask for it (`ls -a`)  

	- Use the TAB key to auto-complete. This is especially helpful if you have to enter a long directory, such as when you ran the `initialize-tsm` command. Note this also works when entering `tsm` commands.  

	- The `time` command helps you get a sense of each step in the process. It can be put in front of ANY shell command. For example:

		`tsm start`  

		`time tsm start`  

		The only difference is, when it's done, it will print the elapsed time to `stdout` (the terminal)  

  - You learned to recognize - and enter - a multi-line command. You do this by ending each line with a backslash (`\`). Example:  

    ```
    ldapsearch -h train-vm \
    -D "uid=admin,o=Users,dc=training,dc=com" \
    -w admin -b "dc=training,dc=com"  
    ```


# References

**Ctrl**-Click or **Cmd**-Click links to open in a new window

* [Get Started with Tableau Server on Linux](https://help.tableau.com/current/server-linux/en-us/get_started_server.htm)  
* [Jump-Start Installation](https://help.tableau.com/current/server-linux/en-us/jumpstart.htm)  
* [Install and Configure Tableau Server](https://help.tableau.com/current/server-linux/en-us/install_config_top.htm)  
* [Driver Download](https://www.tableau.com/support/drivers)  
* [Post Installation Tasks](https://help.tableau.com/current/server-linux/en-us/config_post_install.htm)  
* [tsm-user-identity-store](https://help.tableau.com/current/server/en-us/cli_user-identity_tsm.htm)  
* [Install and Configure Additional Nodes](https://help.tableau.com/current/server-linux/en-us/install_additional_nodes.htm)  
* [Tableau Server Processes](https://help.tableau.com/current/server/en-us/processes.htm)  

# Appendix

## Timings / Planning the Training

Here are some rough timings for steps in the training. Note the **Initialize** step can take up to 15 minutes.

- Install .deb file. **1:30**
- Activate License. **:25**
- Register. **1:10**
- Apply pending changes. **:34**
- Initialize. **14:00**. One option is to present Content Model Best Practices during this time.
- Enable Metadata Services **2:30**

## Registration JSON File  

```
{
  "zip" : "98103",
  "country" : "United States",
  "city" : "Seattle",
  "last_name" : "Biden",
  "industry" : "Software",
  "eula" : "yes",
  "title" : "President and CEO",
  "phone" : "555-123-4567",
  "company" : "Tableau",
  "state" : "Washington",
  "department" : "Sales",
  "first_name" : "Joe",
  "email" : "info@tableau.com"
}

```

## LDAP Identity Store JSON File  

```
{
    "configEntities": {
        "identityStore": {
            "_type": "identityStoreType",
            "type": "activedirectory",
            "domain": "train-vm",
            "root": "",
            "nickname": "",
            "hostname": "train-vm",
            "port": "389",
            "sslPort": "",
            "directoryServiceType": "openldap",
            "bind": "simple",
            "username": "uid=admin,o=Users,dc=training,dc=com",
            "password": "changeme",
            "kerberosPrincipal": "",
            "identityStoreSchemaType": {
                "distinguishedNameAttribute": "",
                "userBaseDn": "o=Users,dc=training,dc=com",
                "userBaseFilter": "objectClass=inetOrgPerson",
                "userUsername": "uid",
                "userClassNames": ["inetOrgPerson"],
                "userDisplayName": "givenName",
                "userEmail": "mail",
                "userCertificate": "",
                "userThumbnail": "",
                "userJpegPhoto": "",
                "memberOf": "member",
                "member": "member",
                "groupBaseDn": "o=Groups,dc=training,dc=com",
                "groupBaseFilter": "objectClass=groupofNames",
                "groupName": "cn",
                "groupEmail": "",
                "groupDescription": "description",
                "serverSideSorting": "false",
                "rangeRetrieval": "false",
                "membersRetrievalPageSize": ""

            }
        }
    }
}
```

## Passwords

| Description | User / ID | Password |
|:------|:---:|:------:|
| Tableau Server Admin | admin | admin |
| node1 (hostname: node-1)  | node1 | node1 |
| node2 (hostname: node-2)  | node2 | node2 |
| train-vm | train | train |
|  SQL Server admin | sa | T@bleau2020|
| SQL Server User<br>This is used when creating a workbook<br><br> | test | T@bleau2021! |
| LDAP Administrator<br> Used with LDAP Utilities<br> (*ldapsearch*, etc.)<br><br> | cn=admin,dc=training,dc=com  | training |


## Terminal Commands  

For your reference, here are the commands you enter via the Terminal

### Install Package

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get -y install gdebi-core
sudo gdebi -n /home/node1/Desktop/tableau-server-2021-3-0_amd64.deb
sudo /opt/tableau/tableau_server/packages/scripts.20213.21.0902.2150/initialize-tsm --accepteula
tsm licenses activate -k <license_key>
tsm register --file /home/node1/Desktop/register.json
```
### LDAP Configuration

```
ldapsearch -h train-vm \
-D "uid=admin,o=Users,dc=training,dc=com" \
-w admin -b "dc=training,dc=com"
cat config.ldap.json

tsm settings import -f /home/node1/Desktop/config.ldap.json
tsm user-identity-store verify-user-mappings -v dev01
tsm user-identity-store verify-group-mappings -v Dev
tsm pending-changes apply
```

### Initialize Server

```
tsm initialize --start-server --request-timeout 1800
tsm maintenance metadata-services enable

tabcmd initialuser \
--username 'admin' \
--password 'admin' \
--server http://localhost
```

### Install SQL Server Driver  

```
sudo apt install unixodbc
wget https://downloads.tableau.com/drivers/linux/deb/tableau-driver/msodbcsql17_17.5.1.1-1_amd64.deb
sudo dpkg -i msodbcsql17_17.5.1.1-1_amd64.deb
```

### Import Group: Command Line  

```
tabcmd login \
--username admin \
--password admin \
--server localhost

tabcmd help syncgroup
tabcmd syncgroup Sales -r Creator
```
### Test SQL server

```
ssh train@train-vm
sqlcmd -U test -S train-vm -P T@bleau2021! -d 'Superstore World'
USE [Superstore World]


sqlcmd -U sa -S train-vm -P T@bleau2020 -d 'Superstore World'
ALTER LOGIN test WITH PASSWORD ='T@bleau2021!'
ALTER LOGIN test WITH CHECK_EXPIRATION=off
```
