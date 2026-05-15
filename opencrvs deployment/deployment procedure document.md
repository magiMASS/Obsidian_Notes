

# Step1: Openshift Deployment

packet receiver 
packet uploader
credential request generator
uin generator
opencrvs mediator
regproc opencrvs stage



# Step: key generation and assignment

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

# Step: key cloak user and client creation
# Step: config file additions and changes

