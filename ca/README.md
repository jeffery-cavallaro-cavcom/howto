# Self-signed Root Certificate Authority (CA)

## Create a New CA

The following procedure is used to construct a self-signed CA cert for the
`cavcom.com` domain.  All server certs should be generated and signed by
this cert.  Note that CRL issues are currently ignored.

1. All steps in this procedure should be performed as root with a strict
   umask (077):
``` bash
sudo -s
umask 077
```
    
1. Select a root directory for the CA:
``` bash
export CA_ROOT=/opt/ca
```

1. Create a directory structure for the new CA:
``` bash
mkdir $CA_ROOT
cd $CA_ROOT
mkdir private index serial certs
```
    
1. Initialize the cert database:
``` bash
touch index/index.txt
echo '01' > serial/serial
```

1. Copy the [root CA configuration file](cavcom.cnf) to the root directory:
   lock it down:
``` bash
cp cavcom.cnf $CA_ROOT
```
    
1. Modify the `dir`, `domain` and `admin` values at the top of the root CA
   configuration file as necessary.
   
1. Lock down the configuration file:
``` bash
cd $CA_ROOT
chmod 400 cavcom.cnf
```

1. Generate a new unencrypted private key and root certificate:
``` bash
openssl req -x509 -config cavcom.cnf -newkey rsa -days 3650 \
    -nodes -out cavcom_cert.pem
```

    If an encrypted key is desired then omit the `-nodes` argument, which will
    result in a prompt for a passphrase.

1. Lock down the public cert and private key:
``` bash
chmod 444 cavcom_cert.pem
chmod 500 private
chmod 400 private/cavcom_key.pem
```

1. View the details of the new public cert to make sure that they are correct:
``` bash
openssl x509 -text -noout -in cavcom_cert.pem
```

## Generate a New Server Cert

1. Create a separate request directory for the new server cert:
``` bash
mkdir -p reqs/test
```

1. Copy the [server request configuration file](reqs/test/test.cnf) to the
   request directory
``` bash
cp reqs/test/test.cnf $CA_ROOT/reqs/test/
```

1. Update the FQDN fields as necessary for the new cert.

1. Generate a new server request:
``` bash
cd $CA_ROOT/reqs/test
openssl req -config test.cnf -new -newkey rsa -days 3650 \
    -nodes -out test_req.pem
```

1. Verify that the new request is correct:
``` bash
openssl req -text -noout -in test_req.pem
```

1. Use the new request to generate a new public server cert:
``` bash
cd $CA_ROOT
openssl ca -config cavcom.cnf -notext -in reqs/test/test_req.pem \
    -out reqs/test/test_cert.pem
```
    
1. Verify that the new cert is correct:
``` bash
openssl x509 -text -noout -in reqs/test/test_cert.pem
```

1. Lock down the new public server cert and private key information:
``` bash
chmod 500 reqs/test
chmod 400 reqs/test/*
```

## Encrypt a Private Key

To encrypt an unencrypted private key:
``` bash
openssl rsa -des -in test_key.pem -out test_key_enc.pem
```

Note that `openssl` will prompt for a passphrase.

## Convert PKCS#8 to PKCS#1

``` bash
openssl rsa -in test_key.pem -out test_key_1.pem
```

## Generate a Random Passphrase

The following is a convenient method for generating random passphrases:
``` bash
openssl rand -base64 12
```

## Create a PKCS12 Keystore

Some applications require a server's private key and public cert to be
packaged into a PKCS12 key file:
``` bash
openssl pkcs12 -export -out test.pfx -inkey test_key_enc.pem -in test_cert.pem
```

Note that most applications assume that the private key is encoded and both
the private key and keystore have a passphrase.

The root signing cert can also be included if required:
``` bash
openssl pkcs12 -export -out test.pfx -inkey test_key_enc.pem \
    -in test_cert.pem -certfile /opt/cert/cavcom_cert.pem
```

## Create a JKS Keystore

Some Java applications require a JKS keystore.  To convert a PKCS12 keystore
to a JKS keystore:
``` bash
keytool -importkeystore -srckeystore test.pfx -srcstoretype pkcs12 \
    -destkeystore test.jks -deststoretype jks
```

To view the results:
``` bash
keytool -list -v -keystore test.jks
```

## Add a Root Cert to the Java Keystore

Root CA certs should be added to the Java default keystore:
``` bash
keytool -import -trustcacerts -alias cavcom \
    -file /var/lib/ca-certificates/local/cavcom_cert.pem \
    -keystore $JAVA_HOME/lib/security/cacerts
```

The default password is `changeit`.
