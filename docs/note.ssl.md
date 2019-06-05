## SSL/TLS 

Assume I know the difference between SSL and TLS. I use SSL as the term for TSL

### concept

PKI - Public Key Infrastructure, this terminalogy is used to imply the whole mechanism of public/private key based framework

Cert - It's wellknown what the public/private keys are. The cert is the public key + some meta data like who issued it, expiration data, server name (or wildcard of server names like *.wikipedia.org), etc.... 

When a client contact server, server reply with a server cert, so that client can encrypt messages that only server will understand (decrypt), then client reply with the client cert (session key/cert) so the server knows how to reply messages encrypted for only this client. 

Cert can be stored in different encoding, namely DEM, JKS, PEM. They can be converted between each other

- Wildcard Certificates - the cert apply to many servers, like *.wikipedia.org
- SubjectAlternativeName(SAN) Certificates - for a group of servers for HA failover purpose

Certificate Signing Request (CSR) vs Certificate - A CSR or Certificate Signing request is a block of encoded text that is given to a Certificate Authority when applying for an SSL Certificate. It is usually generated on the server where the certificate will be installed and contains information that will be included in the certificate such as the organization name, common name (domain name), locality, and country. It also contains the public key that will be included in the certificate. A private key is usually created at the same time that you create the CSR, making a key pair. A CSR is generally encoded using ASN.1 according to the PKCS #10 specification. [link](https://www.sslshopper.com/what-is-a-csr-certificate-signing-request.html)

#### The utility to display server cert
- The following command display all returns from server
  ```
  $ openssl s_client -connect wikipedia.org:443 
  ```
- The following command displays only subject and issuer. x509 is a format standard. -noout means don't display keys
  ```
  $ openssl s_client -connect wikipedia.org:443 | openssl x509 -noout -subject -issuer
  ```
- [command line explain 1](https://wiki.openssl.org/index.php/Command_Line_Utilities)
- [command line explain 2](https://www.sslshopper.com/article-most-common-openssl-commands.html)

- Display Cert expiration dates
  ```
  openssl x509 -in cert.pem -text -noout
  openssl x509 -enddate -noout -in /opt/cloudera/security/pki/$(hostname -f)-server.cert.pem
  ```

#### Convert between cert encodings

Note, DER Encoding is not readable as plain text no matter the file extension of .der .crt .cer.  PEM is readable.

- Convert a DER file (.crt .cer .der) to PEM
  ```
  openssl x509 -inform der -in certificate.cer -out certificate.pem -outform PEM
  ```
  * note:  I recieved this error when tring to convert .cer to .pem
  openssl x509 -inform der -in certnew.cer -out ymcert.pem
  In [this article](https://stackoverflow.com/questions/14191468/openssl-encoding-errors-while-converting-cer-to-pem) and cloudera SSL/TLS guide, just copy to .cer to .pem directly without any convertion needed. 
- Convert a PEM file to DER
  ```
  openssl x509 -outform der -in certificate.pem -out certificate.der
  ```
- Convert a PKCS#12 file (.pfx .p12) containing a private key and certificates to PEM
  ```
  openssl pkcs12 -in keyStore.pfx -out keyStore.pem -nodes
  ```
- You can add -nocerts to only output the private key or add -nokeys to only output the certificates.

- Convert a PEM certificate file and a private key to PKCS#12 (.pfx .p12)
  ```
  openssl pkcs12 -export -out certificate.pfx -inkey privateKey.key -in certificate.crt -certfile CACert.crt
  ```

#### Create CSR
- Create CSR with key, key length is recommanded to be longer than 2048. -keyout is the name for private key. -out is the name for csr
  ```
  openssl req -new -newkey rsa:2048 -nodes -out servername.csr -keyout servername.key
  ```
- Decode a CSR
  ```
  openssl req -in server.csr -noout -text
  ```
- Generate a self-signed certificate 
  ```
  openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt
  ```

- Generate a certificate signing request (CSR) for an existing private key 
  ```
  openssl req -out CSR.csr -key privateKey.key -new
  ```
- Generate a certificate signing request based on an existing certificate 
  ```
  openssl x509 -x509toreq -in certificate.crt -out CSR.csr -signkey privateKey.key
  ```
- Self sign a CSR
  ```
  openssl x509 -req -days 365 -in server.csr -signkey server.key -sha256 -out server.crt
  ```

## Java Keystore and Truststore

(Java) Clients in Cloudera Manager need to check if a received server cert is trusted. It can verify it with default truststore (cacerts) or, alternative truststore, jssecacerts. Inistially we can copy cacerts to jssecacerts so the alternative store contains all the CA information, then add more certs need to be trusted jssecacerts

Note the keystore and truststore may comprise the same file, keystore is for **server** side and truststore is for client side. In Cloudera, each cluster hosts should have its own keystore, or share the same keystore.

Keystore will contain private key, truststore does not

When a server daemon starts, it load keystore. 

- Utility - [keytool](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/keytool.html)


- Convert between JKS and PEM can't happen directly. It need to use both keytoo and openssl, and interim format PKCS12 that both tools understand
  ```
  # from JKS to PEM

  $ keytool -importkeystore -srckeystore /opt/cloudera/security/jks/hostname-keystore.jks  -srcstorepass password -srckeypass password -destkeystore /tmp/hostname-keystore.p12 -deststoretype PKCS12 -srcalias hostname -deststorepass password -destkeypass password

  $ openssl pkcs12 -in /tmp/hostname-keystore.p12 -passin pass:password  -nokeys -out /opt/cloudera/security/pki/hostname.pem
  ```
  ```
  # from PEM to JKS

  $ openssl pkcs12 -export -in /opt/cloudera/security/pki/hostname.pem -inkey /opt/cloudera/security/pki/hostname.key -out /tmp/hostname.p12 -name hostname -passin pass:password -passout pass:password

  $ keytool -importkeystore -srckeystore /tmp/hostname.p12 -srcstoretype PKCS12 -srcstorepass password -alias hostname -deststorepass password -destkeypass password -destkeystore /opt/cloudera/security/jks/hostname-keystore.jks

  ```
  ```
  # Extract private key from pkcs keystore

  $ openssl pkcs12 -in /tmp/hostname-keystore.p12 -passin pass:password -nocerts -out /opt/cloudera/security/pki/hostname.key -passout pass:password

  # Generate key without password

  $ openssl rsa -in /tmp/hostname-keystore.p12 -passin pass:password -nocerts -out /opt/cloudera/security/pki/hostname.pem
  ```
    
#### Add CA to Truststore for TLS/SSL

Many organizations use Windows Domain Controller to distribute internal CA. But for Linux machines, you are on your own. When i tried to use LDAPS to access AD server, the request was rejected because the internal CA as not trusted by my Linux machine. 

I need to let my Java trust my orginizations internal CA. Here is how:

Use openssl to obtain the CA the server is using. In following case, the server is AD server. Copy the outpput begin cert ... end cert to a file called ca.pem
```
openssl s_client -connect server.domain.com:636
```

Covert the pem to der
```
openssl x509 -in ca.pem -inform pem -out ca.der -outform der
```
Validate the der has right content
```
keytool -v -printcert -file ca.der
```

import the der into jssecacerts (java trust store)
```
keytool -importcert -alias org-ad -keystore jssecacerts -storepass changeit -file ca.der
```


Verify the cert is imported
```
keytool -keystore jssecacerts -storepass changeit -list|grep ad
```



#### CA Extension fields

the following content is copied from [this link](https://security.stackexchange.com/questions/100768/difference-between-certificates-with-extension-fields-and-non-repudiation-us)

[Quote]
The extensions are only some kind of policy meta-data attached to the certificate. Technically, no matter the extensions, a certificate remains the same thing: a file linking a key to an identity. Its information could be used for any purpose: authenticating a server, a client, signing other certificates, signing emails, signing software/driver code, etc.

That's where extensions enters into play: they tell the application which usages are applicable to this certificate, so that when the application encounters the certificate in an unauthorized context it could (or must, depending if the usage is set to critical or not) consider the certificate as invalid and refuse it.

However, this check must be done at the application level. A poorly implemented application may not check these usages properly and accept any certificate at any time.

Also, few supplementary things must be noted:

These extension are only restriction. A certificate with no extension is allowed to be used in any role (no usage limitation), while a certificate containing some extension is restricted to the usage defined by these extensions,
Theses extension are actually divided in two main parts (plus one merely hstorical):

The key usage (like "Non-repudiation" in you example) defines lower-level cryptography usage allowed for this certificate,
The extended key usage provides a higher level usage authorized for this certificate ("TLS Web Server Authentication" and "TLS Web Client Authentication" in your examples).
Some applications also handle what's called Netscape Cert Type, it can be seen as the precursor of the extended key usages and gather usages such as "SSL Server", "SSL Client", etc.
All of these key usages may be associated and used in a complementary way.

Some extension might be flagged as "critical": this flag determine how an application not recognizing / having not implemented a specific extension must handle the certificate. Under such situation, if the "critical" flag is enabled the application must reject the certificate, when this flag is not set the application may still accept the certificate.

Extended key usages names (as well as Netscape cert type) are rather straightforward to understand. Key usages however deeply depend on how the protocol (in case of a network communication) will use the certificates. Commonly found key usages for a SSL/TLS client/server application are the following ones:

Server: Digital Signature, Non Repudiation, Key Encipherment,
Client: Digital Signature, Key Encipherment, Data Encipherment.
So, to answer to your edited questions:

Does certificate with "Non Repudiation" usage allow to authenticate all clients with/without "TLS Web Client Authentication" usage?

But "TLS Web Server Authentication" without "Non Repudiation" allows to auth clients only with "TLS Web Client Authentication" usage?

The "Non Repudiation" usage from the server's certificate will be mostly checked by client software, not by the server, and it will have no impact on the way the server will validate client's certificates.

"TLS Web Client Authentication" usage will prevent client's certificates from being used in a server context. It's absence will not block any authentication. However, if a client tries to authenticate using a certificate without this usage but with "TLS Web Server Authentication" instead then the authentication will most likely be refused by the server.

[End Quote]
