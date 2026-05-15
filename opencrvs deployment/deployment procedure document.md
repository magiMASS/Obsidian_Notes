

# Step1: Openshift Deployment

***packet receiver stage
packet uploader stage
credential request generator
uin generator stage
opencrvs mediator
regproc opencrvs stage***



# Step: key generation and Openshift secrets

***keycloak certificate and secrets generation: (openshift cert )

```
-----BEGIN CERTIFICATE-----
MIIDaTCCAlGgAwIBAgIIV5uRrGpxP48wDQYJKoZIhvcNAQELBQAwJjEkMCIGA1UE
AwwbaW5ncmVzcy1vcGVyYXRvckAxNzcwNzI5MTE2MB4XDTI2MDIxMDEzMTE1NloX
DTI4MDIxMDEzMTE1N1owJDEiMCAGA1UEAwwZKi5hcHBzLnVhdC5tc3BzYW5kYm94
LmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMgmgzwTFjsbIJau
kASWjYdIU9x4ns8dTTRzZTZVFW85V2+TRSqUYL1t9ZdhUt3KEPpnT/ABHgJZ+vDF
TuMI7eaV0kCYZ2b32+mgjSTW+VGdCIyhIt/w6MaWWGR/mHz2dUrqSV1TUG8K5LRV
1+OukL2ZXPwByAwVVMnzWZpABCyV6RH9u+7MnH51pcXKDzkUWj9eALM803VxZkRu
2CdHqVXSOsXKUDEt91oDwjvfJO9sU03AfqBze3Sl1S9BID84J1gA/oQamRLqrsIS
6xhYNEiy2jtKwRirnDRA48QZaa5kQojMy+/dBJO6Pk+wKmNfFl1HCRbHRrL6C3vk
cZYB6PECAwEAAaOBnDCBmTAOBgNVHQ8BAf8EBAMCBaAwEwYDVR0lBAwwCgYIKwYB
BQUHAwEwDAYDVR0TAQH/BAIwADAdBgNVHQ4EFgQUg6xkEqbOJhWW3EcbuITIFcWf
nRcwHwYDVR0jBBgwFoAUgkPCxco4xxAzofbRn5nrwmPXuAowJAYDVR0RBB0wG4IZ
Ki5hcHBzLnVhdC5tc3BzYW5kYm94LmNvbTANBgkqhkiG9w0BAQsFAAOCAQEAQKv6
bRiAMCnjSFRlxBYmShCGXsTmgYlB1+J0pIzCGjhReT8//nDBqtgkPSSOLwAmYjc/
ZJHNYzK+QU8OaCq+rNMi1tJA3gukLi8ZwJSOHXoDqnA6+sD3clc8V9GeKEKM2Nc7
E+fA0Mvntjri9cV7NVloAWOSv46XLVgQozeX2lhUGjk39+zZkMQ08Vq8nVJps9dU
JX0mUMWkRty1nmHevgHOLsbwJTG7QAVaWLa2s7nTkxK2RGFfharJai45Ovm8kRtJ
zGiCySLNc9vrQx7SgbqFi0qke7sNY49UbIvHPRwFxWtcwzBeMnKcbIKqLljZtlOI
vm+UB9BFzqLmUq7zWQ==
-----END CERTIFICATE-----


keytool -importcert \
-alias keycloak \
-file keycloak.crt \
-keystore keycloak-truststore.jks \
-storepass Admin@123 \
-noprompt




secret name : keycloak-secret


key : keycloak-truststore.jks

calue : keycloak-truststore.jks

file path: /home/mosip/keycloak

- name: JAVA_TOOL_OPTIONS
    value: "-Djavax.net.ssl.trustStore=/home/mosip/keycloak/keycloak-truststore.jks -Djavax.net.ssl.trustStorePassword=Admin@123"
```

***Opencrvs public and private key generation:

```
create keys public & private :

openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out mosip-priv.key

openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out mosip-priv.key

openssl req -new -x509 -key mosip-priv.key -out opencrvs-pub.key -days 365 -sha512 -subj "/CN=opencrvs"

create cert :

openssl x509 -in opencrvs-pub.key -out opencrvs-cert.pem -outform PEM

 openssl x509 -in pub.pem -pubkey -noout > pub.pem

creatre trust store p12 :

openssl pkcs12 -export \
-out opencrvs-keystore.p12 \
-inkey mosip-priv.key \
-in opencrvs-cert.pem \
-name opencrvs \
-passout pass:Admin@123

```

***Mosip Opencrvs mediator credential  partner certificate generation

```bash
#!/bin/bash

# Clean previous setup (optional but recommended)
rm -rf demoCA *.srl openssl.cnf

# Set up directory structure for the CA
mkdir -p demoCA/newcerts demoCA/private demoCA/certs
touch demoCA/myindex demoCA/serial

# Initialize serial
echo 1000 > demoCA/serial
echo > demoCA/myindex

# Permissions (temporary; tighten later)
chmod 777 demoCA demoCA/private demoCA/certs demoCA/newcerts

# OpenSSL configuration
cat <<EOL > openssl.cnf
[ ca ]
default_ca = CA_default

[ CA_default ]
dir               = ./demoCA
certs             = \$dir/certs
new_certs_dir     = \$dir/newcerts
database          = \$dir/myindex
private_key       = \$dir/private/cakey.pem
serial            = \$dir/serial
default_md        = sha256
policy            = policy_any

[ policy_any ]
commonName = supplied

# Root CA (self-signed)
[ root_cert ]
basicConstraints = CA:TRUE
keyUsage = keyCertSign, cRLSign
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always

# Intermediate CA
[ intermediate_ca ]
basicConstraints = CA:TRUE
keyUsage = keyCertSign, cRLSign
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always

# End entity / Partner cert
[ usr_cert ]
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
EOL

# =========================
# Step 1: Root CA
# =========================
echo "Generating Root CA..."
openssl genpkey -algorithm RSA -out demoCA/private/rootCA.key
openssl req -x509 -new -key demoCA/private/rootCA.key \
  -out demoCA/certs/rootCA.crt \
  -days 3650 \
  -subj "/C=IN/ST=TamilNadu/L=Chennai/O=mpartner-default-opencrvs/CN=OpenCRVS Root CA" \
  -extensions root_cert \
  -config openssl.cnf

# =========================
# Step 2: Intermediate CA
# =========================
echo "Generating Intermediate CA..."
openssl genpkey -algorithm RSA -out demoCA/private/intermediateCA.key
openssl req -new -key demoCA/private/intermediateCA.key \
  -out demoCA/newcerts/intermediateCA.csr \
  -subj "/C=IN/ST=TamilNadu/L=Chennai/O=mpartner-default-opencrvs/CN=OpenCRVS Intermediate CA"

openssl x509 -req \
  -in demoCA/newcerts/intermediateCA.csr \
  -CA demoCA/certs/rootCA.crt \
  -CAkey demoCA/private/rootCA.key \
  -CAcreateserial \
  -out demoCA/certs/intermediateCA.crt \
  -days 3650 \
  -extensions intermediate_ca \
  -extfile openssl.cnf

# =========================
# Step 3: Partner (Leaf)
# =========================
echo "Generating Partner Certificate..."
openssl genpkey -algorithm RSA -out demoCA/private/partnerCA.key
openssl req -new -key demoCA/private/partnerCA.key \
  -out demoCA/newcerts/partnerCA.csr \
  -subj "/C=IN/ST=TamilNadu/L=Chennai/O=mpartner-default-opencrvs/CN=OpenCRVS Partner CA"

openssl x509 -req \
  -in demoCA/newcerts/partnerCA.csr \
  -CA demoCA/certs/intermediateCA.crt \
  -CAkey demoCA/private/intermediateCA.key \
  -CAcreateserial \
  -out demoCA/certs/partnerCA.crt \
  -days 3650 \
  -extensions usr_cert \
  -extfile openssl.cnf

# =========================
# Secure Permissions (FIXED)
# =========================
chmod 600 demoCA/private/*.key
chmod 644 demoCA/certs/*.crt

echo "✅ Certificates generated successfully!"
echo "👉 Root → Intermediate → Partner chain is valid"

# Step: key cloak user and client creation
# Step: config file additions and changes
```


---

# Step: key cloak user and client creation

***Opencrvs user and client create

---

# Step:  PMS create partner & policy 

create partner  mpartner-default-opencrvs
***Upload the partner cert for iopencrvs mosip mediatior to pms


---

# Step: github config update

---
# Step: Db creation and updates

***Create data base : mosip_opencrvs
Create schema : opencrvs
Create Table : birth_transactions, uin_partner_token***


```SQL
-- opencrvs.birth_transactions definition

-- Drop table

-- DROP TABLE opencrvs.birth_transactions;

CREATE TABLE opencrvs.birth_transactions (
	txn_id varchar(64) NOT NULL,
	rid varchar(64) NULL,
	status varchar(2048) NULL,
	cr_by varchar(256) NOT NULL,
	cr_dtimes timestamp NOT NULL,
	upd_by varchar(256) NULL,
	upd_dtimes timestamp NULL,
	is_deleted bool NULL DEFAULT false,
	del_dtimes timestamp NULL,
	access_token varchar NULL,
	opencrvs_brn varchar NULL,
	CONSTRAINT pk_birth_txn_id PRIMARY KEY (txn_id)
);

-- opencrvs.uin_partner_token definition

-- Drop table

-- DROP TABLE opencrvs.uin_partner_token;

CREATE TABLE opencrvs.uin_partner_token (
	uin varchar(50) NOT NULL,
	partner_token varchar(500) NULL,
	cr_by varchar(50) NULL,
	cr_dtimes timestamp NULL,
	upd_by varchar(50) NULL,
	upd_dtimes timestamp NULL,
	partner_name varchar(56) NULL,
	CONSTRAINT uin_partner_token_pkey PRIMARY KEY (uin)
);

```


***Update database

***Data Base : mosip_regprc
Schema : regprc
Table : transaction_type***

```SQL
  INSERT INTO regprc.transaction_type(
  code, descr, lang_code, is_active, cr_by, cr_dtimes, upd_by, upd_dtimes, is_deleted, del_dtimes)
  VALUES
  ('OPENCRVS_NEW', 'OPENCRVS_NEW', 'eng', true, 'MOSIP_SYSTEM', CURRENT_TIMESTAMP, 'some_upd_by_value', CURRENT_TIMESTAMP, false, DEFAULT);
```