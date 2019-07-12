## Kerberos

#### Use AD user other than current login user to operate HDFS

* Use case: I login as root or a local linux user hope to access hdfs. But my user name is not in the HDFS "supergroup". The supergroup is defined as a group in Active Directory. I need to use a AD group member's identity (which I have password) to operate HDFS.

    I need to create a keytab that local user have access to, tell the local user where is the keytab so that the local user can use klist to find the tkt.

    1. create keytab, note the user@realm can be user/fqdn@realm depending on the system configurations

    ```
    # ktutil
    ktutil:  addent -password -p user@realm -k 1 -e aes256-cts-hmac-sha1-96 
    Password for user@realm:
    xxxxxxx
    ktutil:  wkt <filename>.keytab
    ktutil:  q

    # klist -k <filename>.keytab  -Kt //list content of keytab
    ```
    
    2. init keytab so it has the tkt.
    ```
    kinit -p user -kt <filename>.keytab   
    
    klist
    ```
    
    3. test hdfs
    ```
    hdfs dfs -mkdir /user/abc
    ```


#### Query with AD users

* I need to find out the AD user information, for example if it's in certain group, etc. 
  
  I need to find a tool to query such information - it seemed Microsoft is rapidly changing the way to package the tool, the name of the tool, and how to install the tool. 

  1.  find out the windows version number - why windows can't let winver to display both version and bit information with the same tool :
  ```
  winver
  ```
  2.  Install the RSAT - windows 10 changed the way to install it after 1809

  ```
  # Below does not install the needed RSAT.ServerManager.Tools
  dism /online /Get-Features
  dism /online /Enable-feature /FeatureName:DirectoryServices-ADAM-Client
  ```
  4. Use PowerShell with elevated 
  ```
  Get-WindowsCapability -Online | ? Name -like `RSAT.*'
  Get-WindowsCapability -Online | ? Name -like `RSAT.Server*'|Add-WindowsCapability -Online
  ```


  #### None of above works - finally i am trying to install the older version download:
  WindowsTH-RSAT_WS_1803-x64
  after install, type dsa to find "Active Directory Users and Computers"

  https://thewindowscentral.com/active-directory-users-and-computers/

## API

I spent half day trying to get the cm_api to work (need python2, then caused problem with VS Code, Git Bash, etc, etc), only to find out there is a newer better version available:
https://cloudera.github.io/cm_api/docs/python-client-swagger/

It worked. And I also generated the self-signed cert in DER (.cer) format using Chrome, then convert to .PEM format, then the client is able to use it so there's not a warning or error when dealing with this particular self-signed cert.

