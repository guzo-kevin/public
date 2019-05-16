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
The following command display all returns from server
```
$ openssl s_client -connect wikipedia.org:443 
```
The following command displays only subject and issuer. x509 is a format standard. -noout means don't display keys
```
$ openssl s_client -connect wikipedia.org:443 | openssl x509 -noout -subject -issuer
```
- [command line explain 1](https://wiki.openssl.org/index.php/Command_Line_Utilities)
- [command line explain 2](https://www.sslshopper.com/article-most-common-openssl-commands.html)

#### Convert between cert encodings
- Convert a DER file (.crt .cer .der) to PEM
  ```
  openssl x509 -inform der -in certificate.cer -out certificate.pem
  ```
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


