## SSL/TLS

#### Self-signed certs

The basic idea of using self-signed certs in Cloudera is:

https://www.cloudera.com/documentation/enterprise/5-14-x/topics/sg_self_signed_tls.html

* use keytool to create a self-signed cert, store it in jks
```
keytool -genkeypair -alias cmhost -keyalg RSA -keysize 2048 -dname "cn=cm01.example.com, ou=Department, o=Company, l=City, st=State, c=US" -keypass password -keystore example.jks -storepass password
```

* use keytool to export the cert to .cer (.pem) file
(it may need to use both keytool and openssl https://www.cloudera.com/documentation/enterprise/5-14-x/topics/cm_sg_openssl_jks.html )

```
 keytool -export -alias cmhost -keystore example.jks -rfc -file selfsigned.pem
```

* use keytool to import the .cer to jssecacerts trust store
```
keytool -importcert -alias xxxx -keystore jssecacerts -file selfsigned.pem -storepass changeit
```
* let CM know where the keystore and trust store files are, and which key to use

* A private key (.pem), password file (.pw) and a cert (.pem) are needed for agent. 
```
in /etc/cloudera-scm-agent/config.ini

client_key_file=/opt/cloudera/security/x509/agent.key
client_keypw_file=/opt/cloudera/security/x509/agentkey.pw 
client_cert_file=/opt/cloudera/security/x509/keyforall.pem

to get private key from keystore:

keytool -importkeystore -srckeystore key-keystore.jks \
-srcstorepass password -srckeypass password \
-destkeystore agentkey.p12 \
-deststoretype PKCS12 -srcalias keyforall -deststorepass \
password -destkeypass password

openssl pkcs12 -in agentkey.p12  -passin pass:password -nocerts -out  agent.key -passout pass:password
```


#### Root and Intermediate CA
https://www.cloudera.com/documentation/enterprise/5-14-x/topics/sg_add_root_ca_explicit_trust.html

* root ca goes to trust store
* intca append to host.cert.pem

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
    
    1. init keytab so it has the tkt.
    ```
    kinit -p user -kt <filename>.keytab   
    
    klist
    ```
    
    1. test hdfs
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

  [https://thewindowscentral.com/active-directory-users-and-computers/](https://cloudera.github.io/cm_api/docs/python-client-swagger/)


## Query for AD User for LDAP

#### Linux
```
ldapsearch -v -H ldap://adserver.abc.com -D'abc@abc.com' -w'abcpassword' -b'OU=team,OU=Office,OU=users,dc=abc,dc=com'
# the above command query everything under the OU

ldapsearch -v -H ldap://adserver.abc.com -D'abc@abc.com' -w'abcpassword' -b'CN=group_name,OU=groups,OU=team,OU=Office,OU=users,dc=abc,dc=com'
# the above command query everything in the CN group
```
#### Windows
```
Get-ADUser -Filter 'SamAccountName -like "*admin"'
Get-ADUser -Filter 'Name -like "*admin"'
Get-ADGroup -Filter 'Name -like "*admin*"'
```

## API

I spent half day trying to get the cm_api to work (need python2, then caused problem with VS Code, Git Bash, etc, etc), only to find out there is a newer better version available:
[https://cloudera.github.io/cm_api/docs/python-client-swagger/](https://cloudera.github.io/cm_api/docs/python-client-swagger/)
or
[https://archive.cloudera.com/cm6/6.2.0/generic/jar/cm_api/swagger-html-sdk-docs/python/docs/ServicesResourceApi.html#read_services](https://archive.cloudera.com/cm6/6.2.0/generic/jar/cm_api/swagger-html-sdk-docs/python/docs/ServicesResourceApi.html#read_services)

and this ultimate doc: [https://archive.cloudera.com/cm6/6.2.0/generic/jar/cm_api/swagger-html-sdk-docs/python/README.html](https://archive.cloudera.com/cm6/6.2.0/generic/jar/cm_api/swagger-html-sdk-docs/python/README.html)
It worked. And I also download/save the self-signed cert in DER (.cer) format using Chrome, then convert to .PEM format, then the client is able to use it so there's not a warning or error when dealing with this particular self-signed cert.

## LDAP
DN = Distinguished name, the full path to the object
CN = common name 
OU = folder name, from children to parents, for example, ou=child_folder, ou=parent_folder
dc = domain, dc=abc,dc=com

