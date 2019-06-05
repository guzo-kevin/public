## Auto TLS

|Filename|Description|
|:-------:|:---------|
|cm-auto-global_cacerts.pem|CA certificate and other trusted certificates in PEM format|
|cm-auto-global_truststore.jks|CA certificate and other trusted certificates in JKS format|
|cm-auto-in_cluster_ca_cert.pem|CA certificate in PEM format|
|cm-auto-in_cluster_truststore.jks|CA certificate in JKS format|
|cm-auto-host_key_cert_chain.pem|Agent host certificate and private key in PEM format|
|cm-auto-host_cert_chain.pem|Agent host certificate in PEM format|
|cm-auto-host_key.pem|Agent host private key in PEM format|
cm-auto-host_keystore.jks|Agent host private key in JKS format|
|cm-auto-host_key.pw|Agent host private key password file|


## Manually configure TLS

#### for User to access the CM Server with TLS:

|Parameter|File location|
|:----|:----|
| Cloudera Manager TLS Server JKS Keystore |  /opt/cloudera/security/pki/server.jks |
| server jks passwd |  password for server.jks |
| Use TLS encryption for admin console |  check the box |


#### For CM services to access back to CM Server with TLS

|Parameter|File location|
|:----|:----|
| client trust store location |  <JAVA_HOME>/jre/lib/security/jssecacerts |
| server trust store password |  password for jssecacerts |


#### For Agents to access CM Server with TLS

|Parameter|File location|
|:----|:----|
| Use TLS for Agent  |  Enable check box |
| Edit /etc/cloudera-scm-agent/config.ini  |  use_tls =1 |
| Edit /etc/cloudera-scm-agent/config.ini  |  set verify_cert_file to path_to_pem |

**Note** : need to use config.ini from any nodes other than CM. Because on CM the server_host  is set to localhost, on other nodes the server_host is set to the server name where CM resides. 


#### For CM authenticate agent with TLS


- export JAVA_HOME=/usr/java/jdk1.8.0_141-cloudera
- mkdir -p /opt/cloudera/security/pki
- use keytool to generate keystore (which store a cert/privatekey/publickey), store the keystore with name $(hostname -f).jks in /opt/cloudera/security/pki
- sign the csr (if it's for internal CA)
- the signed PEM (text file) should be stored in /opt/cloudera/security/pki/$(hostname -f).pem
- copy root cert to /opt/cloudera/security/pki/rootca.pem
- copy intermediate ca cert to /opt/cloduera/security/pki/intca.pem (intca-1.pem, intca-2.pem if there are multiple intermediate CA certs)
- create $JAVA_HOME/jre/lib/security/jssecacerts with appropriate contents
- import rootca.pem into jssecacerts
  ```
  keytool -importcert -alias rootca -keystore $JAVA_HOME/jre/lib/security/jssecacerts -file /opt/cloudera/security/pki/rootca.pem
  ```

- append all intca.pem to $(hostname -f).pem, then import into $(hostname -f).jks 
  ```
  cat /opt/cloudera/security/pki/intca.pem >> /opt/cloudera/security/pki/$(hostname -f).pem

  keytool -importcert -alias $(hostname -f) -file /opt/cloudera/security/pki/$(hostname -f).pem -keystore /opt/cloudera/security/pki/$(hostname -f).jks
  ```

- on all agent hosts: ln -s /opt/cloudera/security/pki/$(hostname -f).pem /opt/cloudera/security/pki/agent.pem. This is to allow the same /etc/cloudera-scm-agent/config.ini file for all agent hosts.
- on CM Serfer: ln -s /opt/cloudera/security/pki/$(hostname -f).jks /opt/cloudera/security/pki/server.jks
  

