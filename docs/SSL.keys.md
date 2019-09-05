# Keys

* ssh
   - id_rsa
   - id_ecdsa
   - *.pub

* OpenSSl
   - priv.key
   - pubkey.pem
   - key.der
   - example.com.crt
   - cert.pem (cert)
   - csr.pem (intermediary request for cert)
   - fullchain.pem (when include authority chains)

* formats

   - PEM is simply a base64 encoded DER file (with a plain-text header and footer)
   - DER is the binary format of ASN.1, more or less a dark-ages BSON
   - (you colud say DER is to UTF-8 as ASN.1 is to Unicode)
   - ASN.1 would then be a sort of dark-ages object notation for anything
   - X.509, then, is a dark-ages schema that uses ASN.1 to describe a specefic, concrete thing

# SSH Public Key  (RFC4253)

Referenced [here](https://coolaj86.com/articles/the-ssh-public-key-format/)

```
[type-name] [base64-encoded-ssh-public-key] [comment]

ssh-rsa AAAAB3NzaC1yc2E...Q02P1Eamz/nT4I3 root@localhost

```

* public key finger print (RFC1321)

   The security of the SSH protocols relies on the verification of
   public host keys.  Since public keys tend to be very large, it is
   difficult for a human to verify an entire host key.  Even with a
   Public Key Infrastructure (PKI) in place, it is useful to have a
   standard for exchanging short fingerprints of public keys.

   for example:
   ```
   c1:b1:30:29:d7:b8:de:6c:97:77:10:d7:46:41:63:87
   ```

# SSH Private Key

OpenSSH and OpenSSL used to use the same format for private key. But OpenSSH started to use its own format. 

* OpenSSL Key is like this:

```
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgiYydo27aNGO9DBUW
eGEPD8oNi1LZDqfxPmQlieLBjVShRANCAAQhPVJYvGxpw+ITlnXqOSikCfz/7zms
yODIKiSueMN+3pj9icDgDnTJl7sKcWyp4Nymc9u5s/pyliJVyd680hjK
-----END PRIVATE KEY-----

or 

-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAxhhbENu3NytrsWSUweH0y+UqI3JtzzzzoV0QqQm2CfikMR6p  

-----END RSA PRIVATE KEY-----
```


* New OpenSSh key is like this:
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAaAAAABNlY2RzYS
1zaGEyLW5pc3RwMjU2AAAACG5pc3RwMjU2AAAAQQR9WZPeBSvixkhjQOh9yCXXlEx5CN9M
yh94CJJ1rigf8693gc90HmahIR5oMGHwlqMoS7kKrRw+4KpxqsF7LGvxAAAAqJZtgRuWbY
EbAAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBH1Zk94FK+LGSGNA
6H3IJdeUTHkI30zKH3gIknWuKB/zr3eBz3QeZqEhHmgwYfCWoyhLuQqtHD7gqnGqwXssa/
EAAAAgBzKpRmMyXZ4jnSt3ARz0ul6R79AXAr5gQqDAmoFeEKwAAAAOYWpAYm93aWUubG9j
YWwBAg==
-----END OPENSSH PRIVATE KEY-----

And its content include: (credit to: https://coolaj86.com/articles/the-openssh-private-key-format/ )

"openssh-key-v1"0x00    # NULL-terminated "Auth Magic" string
32-bit length, "none"   # ciphername length and string
32-bit length, "none"   # kdfname length and string
32-bit length, nil      # kdf (0 length, no kdf)
32-bit 0x01             # number of keys, hard-coded to 1 (no length)
32-bit length, sshpub   # public key in ssh format
    32-bit length, keytype
    32-bit length, pub0
    32-bit length, pub1
32-bit length for rnd+prv+comment+pad
    64-bit dummy checksum?  # a random 32-bit int, repeated
    32-bit length, keytype  # the private key (including public)
    32-bit length, pub0     # Public Key parts
    32-bit length, pub1
    32-bit length, prv0     # Private Key parts
    ...                     # (number varies by type)
    32-bit length, comment  # comment string
    padding bytes 0x010203  # pad to blocksize (see notes below)
```

# Putty Key (.ppk)

ppk key contain both public and private keys. 

# ASN.1 and  
https://coolaj86.com/articles/openssh-vs-openssl-key-formats/

https://coolaj86.com/articles/asn1-for-dummies/

https://coolaj86.com/articles/csr-my-old-friend/

https://coolaj86.com/articles/the-openssh-private-key-format/

https://asn1tools.readthedocs.io/en/latest/

# python example -  use python to read .ssh/id_rsa
http://snmplabs.com/pyasn1/example-use-case.html#read-your-ssh-id-rsa


# save and process RSA key: 
https://stackoverflow.com/questions/45146504/python-cryptography-module-save-load-rsa-keys-to-from-file

save/load with 509 https://cryptography.io/en/latest/x509/reference/

https://cryptography.io/en/latest/hazmat/primitives/asymmetric/serialization/

test with "is_instance" https://www.programcreek.com/python/example/99670/cryptography.hazmat.primitives.asymmetric.rsa.RSAPublicKey

https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/

different library:

https://www.pyopenssl.org/en/stable/api/crypto.html#certificates

# read content of key
```
openssl x509 -in cacert.pem -text -noout

keytool -printcert -file selfsigned.cer 
```