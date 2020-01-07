## SSL/TLS


## What is the goal
1. Need to access the console with https//SSL/TLS (port 7183 vs 7180)
2. Need to encrypt the traffic between agent and server with TLS/SSL. But any cert would be ok
3. Need to authenticate server and agent so only right certs can be used between them 
4. TLS/SSL for cloudera clusters (services)

## How it works

A listening (server) component need to provide a public cert to client component, it also need a key so it can decrypt the client message encrypted using the cert. Client need to trust the cert that server component provided. 

So we are talking about 3 things. Public Cert, Private Key, and Truststore. Private key and truststore need to be protected by password. so there are 2 or more passwords. Public/Private key can be provided in 2 formats, jks and pem for different components.

In cloudera manager, only agent uses pem format of pub/pri keys. other components need to use jks which stores both pub and pri certs. 

How to make a cert trusted? for most components jssecacert are used as truststore. For cm agent, a trusted cert is specified in /etc/cloudera-cm-agent/config.ini, either to specify the self-signed cert, or to specify a root cert, root + intimediate certs.

All the agent configuration is in config.ini. All the other configurations are in scm database. 

## what we need
* jssecert with all certs added
* a pem with all root ca + intemidiate certs, or self-signed cert for agent to trust
* a pem of cert (or self-signed cert) for agent to use
* a pem of private key and password for agent to use
* a jks with both cert and key for cm server to use, there should be only one cert and key in the jks because I did not see alias can be specified 

## Config CM

* server configuration

```
  /cm/config (Administration -> Settings -> security):
    KEYSTORE_PATH (cm step1)
    KEYSTORE_PASSWORD (cm step1)
    WEB_TSL (cm step1) # once this switch flips the cm server will try to start using TLS
    TRUSTORE_PATH (agent step4)
    TRUSTORE_PASSWORD (agent step4)

  /cm/service/config (Services -> Configuration -> Scope (Service-Wide) -> Security )
    ssl_client_truststore_location (cm step2)
    ssl_client_truststore_password (cm step2)
    Note: the display name for above 2 parameters were wrong on document. 

  /cm/config (Administration -> Settings -> security):
    AGENT_TLS (agent step1) # once this set cm start use TLS to communicate with agent
    NEED_AGENT_VALIDATION (agent step4) # once this set cm verify if agent is using right cert (in trust store and if the cert actually is valid for the server (DN or SAN))
  
  I've not verified Whether following is needed 
    ssl_server_keystore_password
    ssl_server_keystore_location
    in
    /cm/service/roleConfigGroups/mgmt-ACTIVITYMONITOR-BASE/config
    /cm/service/roleConfigGroups/mgmt-HOSTMONITOR-BASE/config
    /cm/service/roleConfigGroups/mgmt-SERVICEMONITOR-BASE/config
    /cm/service/roleConfigGroups/mgmt-NAVIGATORMETASERVER-BASE/config 

```
## Config agent


* agent configuration (config.ini)
```
  use_tls=1 (agent step2)
  verify_cert_file: specify if the self-signed cert or root/intemediate certs to trust
  client_key_file: private key in pem format
  client_cert_file: public cert in pem format
  client_keypw_file: clear text password protected by linux file permission (440 by root)
```


##  Reverse configuration
You can update the CONFIGS table in scm database to reverse the web_tls or agent_tls in case things do not work out as expected, other settings such as the passwords, locations of keystore and trust store, does not matter.  

select * from CONFIGS where ATTR='agent_tls';
select * from CONFIGS where ATTR='web_tls';

## Cert - Key convertion

I have experience 2 scinarious need to convert between the format

1. start from self-signed cert 

* i used keytool to generate self-signed cert, it stores cert/key in jks
```
keytool -genkeypair -alias keyforall -keyalg RSA -keysize 2048 -dname "cn=abc.com, ou=abc, o=dep, l=DC, st=DC, c=US" -keypass password! -keystore keyforall-keystore.jks -storepass password! -validity 365 -ext SAN=dns:abc1,dns:abc2,dns:abc1.com,dns:abc2.com
```
* use keytool to export cert to pem (cert only)
```
keytool -export -alias keyforall -keystore keyforall-keystore.jks -storepass password! -rfc -file keyforall.pem
```

* use keytool to retrieve private key to p12 (destkeystore)
```
keytool -importkeystore -srckeystore keyforall-keystore.jks \
         -srcstorepass password! -srckeypass password! \
         -destkeystore agentkey.p12 \
         -deststoretype PKCS12 -srcalias keyforall -deststorepass \
         password! -destkeypass password!
```

* use openssl to convert private key p12 to key in pem format (with password encryption)

```
openssl pkcs12 -in agentkey.p12 -passin pass:password! -nocerts -out agent.key -passout pass:password!
```

2. start from CA signed cert 
* i received cert/key in .pfx format (exported from windows cert store), .pfx is a type of pkcs12
* i used openssl to retrieve all certs, key, client cert, cacerts respectively in pem format
```
winpty openssl pkcs12 -in abc.pfx -out abc.all.pem -nodes
winpty openssl pkcs12 -in abc.pfx -out abc.pri.pem -nocerts -nodes
winpty openssl pkcs12 -in abc.pfx -out abc.pub.pem -nokeys -clcerts -nodes
winpty openssl pkcs12 -in abc.pfx -out abc.ca.pem -nokeys -cacerts -nodes
```
* i used keytool to import client (pub) cert, cacerts, i manually split cacert to root and int,  to jssecacert
```
keytool -importcert -alias root_ca -keystore jssecacerts -file ca.root.pem -noprompt -storepass changeit
keytool -importcert -alias int1 -keystore jssecacerts -file ca.int1.pem -noprompt -storepass changeit
keytool -importcert -alias int2 -keystore jssecacerts -file ca.int2.pem -noprompt -storepass changeit
keytool -importcert -alias client_cert -keystore jssecacerts -file abc.pub.pem -noprompt -storepass changeit
```
* i used openssl to convert all.pem to p12 (for some reason the .pfx does not work as p12 in some situations)
```
openssl pkcs12 -export -in abc.all.pem -name abc-all -passout pass:actual_password > abc.all.p12
```
* use keytool to import the p12 to a jks
```
keytool -importkeystore  -deststorepass actual_password -destkeystore abc-keystore.jks -srckeystore abc.all.p12  -srcstoretype PKCS12 -srcstorepass actual_password
```
* use keytool to import the p12 to a jks



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

Here is a very clear version of API doc
[https://archive.cloudera.com/cm6/6.0.0/generic/jar/cm_api/apidocs/index.html](https://archive.cloudera.com/cm6/6.0.0/generic/jar/cm_api/apidocs/index.html)

## LDAP
DN = Distinguished name, the full path to the object
CN = common name 
OU = folder name, from children to parents, for example, ou=child_folder, ou=parent_folder
dc = domain, dc=abc,dc=com

