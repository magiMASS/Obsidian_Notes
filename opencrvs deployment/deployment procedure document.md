# Step1: Db creation and updates

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








# Step 2 : key generation and Openshift secrets

***keycloak certificate and secrets generation: (openshift cert )

get the route cert from devops root and inter ca
```cert
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
```

```bash
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

```bash
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

---



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

# Step 3: key cloak user and client creation

***Create Opencrvs user and client
Client : 
		mpartner-default-opencrvs
user :
		mpartner-default-opencrvs
Add roles :  
		SUBSCRIBE_CREDENTIAL_ISSUED_INDIVIDUAL
		PUBLISH_CREDENTIAL_STATUS_UPDATE_GENERAL

---
# Step 4: Git hub config update

***Add in partner-management-mz.properties
```GIT
#Allowed credential types which partner can map against to policy
pmp.allowed.credential.types=auth,qrcode,euin,reprint,opencrvs
```

***Add in application-mz.properties
```GIT

packetmanager.default.priority=source:CNIE\/process:CORRECTION,source:REGISTRATION_CLIENT\/process:NEW|UPDATE|LOST,source:OPENCRVS\/process:OPENCRVS_NEW|OPENCRVS_DEATH,source:RESIDENT\/process:ACTIVATED|DEACTIVATED|RES_UPDATE|RES_REPRINT

provider.packetreader.opencrvs=source:OPENCRVS,process:OPENCRVS_NEW|OPENCRVS_DEATH,classname:io.mosip.commons.packet.impl.PacketReaderImpl

provider.packetwriter.opencrvs=source:OPENCRVS,process:OPENCRVS_NEW|OPENCRVS_DEATH,classname:io.mosip.commons.packet.impl.PacketWriterImpl

websub.base.url=http://websub-service.mosip.svc.cluster.local:9191
```

***Add in registration-processor-camel-routes-opencrvs_new-mz.xml

```XML
<routes xmlns="http://camel.apache.org/schema/spring">
    <!--   packet-receiver&ndash to securezone-notification-->
    <route id="packet-receiver-->securezone-notification opencrvs_new route">
        <from uri="eventbus://packet-receiver-opencrvs_new-bus-out"/>
        <log message="packet-receiver-->securezone-notification opencrvs_new ${bodyAs(String)}"/>
        <choice>
            <when>
                <simple>${bodyAs(String)} contains '"internalError":true'</simple>
                <to uri="eventbus:retry-bus-in"/>
            </when>
            <when>
                <simple>${bodyAs(String)} contains '"isValid":false'</simple>
                <to uri="eventbus:error-bus-in"/>
            </when>
            <otherwise>
                <!--            <to uri="eventbus:securezone-notification-bus-in" />-->

                <simple>${bodyAs(String)} contains '"isValid":true'</simple>
                <process ref="tokenGenerationProcessor"/>
                <setHeader headerName="CamelHttpMethod">
                    <constant>POST</constant>
                </setHeader>
                <setHeader headerName="Content-Type">
                    <constant>application/json</constant>
                </setHeader>
                <setHeader headerName="Cookie">
                    <simple>${header.Cookie}</simple>
                </setHeader>
                <setBody>
                    <simple>${bodyAs(String)}</simple>
                </setBody>
                
                <to uri="http://regproc-securezone-notification-stage.mosip.svc.cluster.local/registrationprocessor/v1/securezone/notification"/>

            </otherwise>
        </choice>
    </route>
    <!--   securezone-notification  to packet-uploader-->
    <route id="securezone-notification-->packet-uploader opencrvs_new route">
        <from uri="eventbus:securezone-notification-opencrvs_new-bus-out"/>
        <log message="securezone-notification-->packet-uploader opencrvs_new route ${bodyAs(String)}"/>
        <choice>
            <when>
                <simple>${bodyAs(String)} contains '"isValid":true'</simple>
                <to uri="eventbus:packet-uploader-bus-in"/>
            </when>
            <when>
                <simple>${bodyAs(String)} contains '"internalError":true'</simple>
                <to uri="eventbus:retry-bus-in"/>
            </when>
            <otherwise>
                <to uri="eventbus:error-bus-in"/>
            </otherwise>
        </choice>
    </route>
    
    
    <route id="packet-uploader route-->uin-generator opencrvs_new route">
        <from uri="eventbus:packet-uploader-opencrvs_new-bus-out"/>
        <log message="packet-uploader route-->uin-generator opencrvs_new route ${bodyAs(String)}"/>
        <choice>
            <when>
                <simple>${bodyAs(String)} contains '"internalError":true'</simple>
                <to uri="eventbus:retry-bus-in"/>
            </when>
            <when>
                <simple>${bodyAs(String)} contains '"isValid":false'</simple>
                <to uri="eventbus:error-bus-in"/>
            </when>
            <otherwise>
                <to uri="eventbus:uin-generator-bus-in"/>
            </otherwise>
        </choice>
    </route>
    <route id="uin-generator-->finalization opencrvs_new route">
        <from uri="eventbus:uin-generator-opencrvs_new-bus-out"/>
        <log message="uin-generator-->opencrvs opencrvs_new route ${bodyAs(String)}"/>
        <choice>
            <when>
                <simple>${bodyAs(String)} contains '"internalError":true'</simple>
                <to uri="eventbus:retry-bus-in"/>
            </when>
            <when>
                <simple>${bodyAs(String)} contains '"isValid":false'</simple>
                <to uri="eventbus:error-bus-in"/>
            </when>
            <otherwise>
                <to uri="eventbus:opencrvs-bus-in"/>
            </otherwise>
        </choice>
    </route>
</routes>
```


***Add in registration-processor-mz.properties

```GIT
#Route files corresponding to the secure flow
camel.secure.active.flows.file.names=registration-processor-camel-routes-new-mz.xml,registration-processor-camel-routes-update-mz.xml,registration-processor-camel-routes-activate-mz.xml,registration-processor-camel-routes-res-update-mz.xml,registration-processor-camel-routes-deactivate-mz.xml,registration-processor-camel-routes-lost-mz.xml,registration-processor-camel-routes-res-reprint-mz.xml,registration-processor-camel-routes-opencrvs_new-mz.xml

#for open crvs stage
packetmanager.provider.packetvalidator.IDSchemaVersion=source:OPENCRVS/process:OPENCRVS_NEW|OPENCRVS_DEATH

#opencrvs-stage
mosip.regproc.opencrvs.eventbus.kafka.commit.type=single
mosip.regproc.opencrvs.eventbus.kafka.max.poll.records=100
mosip.regproc.opencrvs.eventbus.kafka.max.poll.interval=900000
mosip.regproc.opencrvs.eventbus.kafka.poll.frequency=100
mosip.regproc.opencrvs.eventbus.kafka.group.id=opencrvs-stage
mosip.regproc.opencrvs.message.expiry-time-limit=${mosip.regproc.common.stage.message.expiry-time-limit}
mosip.regproc.opencrvs.server.port=8045
mosip.regproc.opencrvs.server.servlet.path=/registrationprocessor/v1/opencrvs-stage
mosip.regproc.opencrvs.eventbus.port=5745
mosip.regproc.opencrvs.credentialtype=opencrvs
mosip.regproc.opencrvs.issuer=mpartner-default-opencrvs
#mosip.regproc.opencrvs.issuer=opencrvs-partner


registration.processor.main-processes=NEW,UPDATE,LOST,RES_UPDATE,ACTIVATE,DEACTIVATE,OPENCRVS_NEW

mosip.registration.processor.credential.partner-profiles=registration-processor-credential-partners.json
mosip.registration.processor.credential.default.partner-ids=digitalcardPartner,opencrvsPartner
mosip.registration.processor.credential.conditional.partner-id-map={'printPartner':'{"14023"} contains postalCode'}
mosip.registration.processor.credential.conditional.no-match-partner-ids=printPartner

```

***Add in opencrvs-mz.properties

```GIT
# Following properites have their values assigned via 'overrides' environment variables of config server docker.
# DO NOT define these in any of the property files.  They must be passed as env variables.  Refer to config-server
# helm chart:
keycloak.internal.url=https://keycloak-default.apps.uat.mspsandbox.com
# Following properties get their values from environment variables of mediator helm chart.
# DO NOT define the following properties in this file.
db.dbuser.password=YJEJZbXMMbiDWSEconeCnA
opencrvs.receive.credential.url=http://10.10.10.217:2024/birthReceiveNid
mosip.receive.credential.url=http://my-release-opencrvs-mediator.default.svc.cluster.local:80/opencrvs/v1/internal/receiveCredentialBirth
mosip.opencrvs.websub.resubscribe.lease.seconds=864000
opencrvs.auth.url=
opencrvs.locations.url=
opencrvs.client.id=
opencrvs.client.id=
opencrvs.client.secret.key=
opencrvs.client.sha.secret=
#mosip.opencrvs.client.id=mosip-opencrvs-client
#mosip.opencrvs.client.secret.key=
mosip.api.internal.host=apps.uat.mspsandbox.com
#mosip.opencrvs.partner.client.id=opencrvs-partner
#mosip.opencrvs.partner.client.sha.secret=
#mosip.opencrvs.partner.username=opencrvs-partner
#mosip.opencrvs.partner.password=
mosip.opencrvs.uin.token.partner=mpartner-default-opencrvs
#mosip.opencrvs.death.client.id=mosip-opencrvs-client
#mosip.opencrvs.death.client.secret=
mosip.resident.client.secret=Iu1Phoref2SvrYtvHSLd07YKQXx9aQ04
mosip.idrepo.client.secret=x8X3WPx6kQA6u4O3rYCDV8qndpZX1YAd

opencrvs.keystore.password=Admin@123



openapi.service.server.url=${server.servlet.context-path}

mosip.opencrvs.client.id=mosip-resident-client  
mosip.opencrvs.client.secret.key=${mosip.resident.client.secret}
mosip.opencrvs.death.client.id=mosip-idrepo-client
mosip.opencrvs.death.client.secret=${mosip.idrepo.client.secret} 
mosip.opencrvs.partner.client.id=mpartner-default-opencrvs
mosip.opencrvs.partner.client.secret=uZA4OHO9nwJ9X7OENbdbCI9GF9QSMWVb
mosip.opencrvs.partner.client.sha.secret=mpartner-default-opencrvs
mosip.opencrvs.partner.username=mpartner-default-opencrvs
mosip.opencrvs.partner.password=Admin@123
mosip.opencrvs.uin.token.partner=mpartner-default-opencrvs
mediator.core.pool.size=20
mediator.max.pool.size=200
mediator.queue.capacity=50
opencrvs.datetime.pattern=yyyy-MM-dd'T'HH:mm:ss.SSS'Z'
opencrvs.center.id=45451
opencrvs.machine.id=45452
opencrvs.appid=opencrvs
opencrvs.appName=OPENCRVS
opencrvs.audit.app.id=${opencrvs.appid}
opencrvs.audit.app.name=${opencrvs.appName}
opencrvs.data.lang.code.default=eng
opencrvs.data.lang.code.mapping=eng:eng|english|en,fra:french|fr|fra|fre,ara:ara|arabic|ar
opencrvs.data.address.line.mapping=1:1-2|2:2-3|3:3-4
opencrvs.data.address.line.joiner=", "
region:state:id|zone:district:id|city:city:direct     id means call location url , direct means use as it is
opencrvs.data.address.location.mapping=region:state:direct|zone:district:direct|city:city:direct
opencrvs.data.dummy.phone="9898989898"
opencrvs.data.dummy.emailSuffix=".123@gmail.com"
# The following process.type should the same one present in provider.packetwriter.opencrvs and provider.packetreader.opencrvs, in application-default.properties
opencrvs.birth.process.type=OPENCRVS_NEW
# Incase the mediator encounters error creating and uploading the packet, it will reproduce the same if this is true.
opencrvs.reproduce.on.error=false
opencrvs.reproduce.on.error.delay.ms=10000
mosip.opencrvs.websub.resubscribe=true
mosip.opencrvs.websub.resubscribe.init.delay.ms=20000
mosip.opencrvs.websub.resubscribe.delay.ms=21600000
mosip.opencrvs.privkey.path=/home/mosip/certs/mosip-priv.key
#priv key of opencrvs-cred-partner
#mosip.opencrvs.privkey.path=/home/mosip/certs/credkeystore.p12   #priv key of opencrvs-cred-partner
opencrvs.mosip.pubkey.path=/home/mosip/certs/opencrvs-pub.key
#cert of opencrvs-auth-partner
#opencrvs.mosip.pubkey.path=/certs/mnt/authkeystore.p12  #cert of opencrvs-auth-partner
kernel.auth.adapter.available=false
mosip.iam.token_endpoint=${keycloak.internal.url}/auth/realms/mosip/protocol/openid-connect/token
mosip.iam.validate_endpoint=${keycloak.internal.url}/auth/realms/mosip/protocol/openid-connect/userinfo
config.server.file.storage.uri=http://config-server.config-server/config/*/default/${spring.cloud.config.label}/
registration.processor.identityjson=identity-mapping.json
mosip.registration.processor.registration.sync.id=mosip.registration.sync
mosip.registration.processor.application.version=1.0
#MIDSCHEMAURL=${mosip.kernel.syncdata-service-idschema-url}
#SYNCSERVICE=${mosip.regproc.status.service.url}/registrationprocessor/v1/registrationstatus/sync
#PACKETRECEIVER=${mosip.packet.receiver.url}/registrationprocessor/v1/packetreceiver/registrationpackets
#RIDGENERATION=http://kernel-ridgenerator-service.mosip.svc.cluster.local/v1/ridgenerator/generate/rid
#KEYMANAGER_TOKENID=${mosip.kernel.keymanager.url}/v1/keymanager/tokenid
#IDREPO_IDENTITY=${mosip.idrepo.identity.url}/idrepository/v1/identity/
#IDREPO_VID=${mosip.idrepo.vid.url}/idrepository/v1/vid
IDSchema.Version=0.1
id.repo.update=mosip.id.update
objectstore.crypto.name=OnlinePacketCryptoServiceImpl
objectstore.adapter.name=PosixAdapter
object.store.base.location=./packets/mosip-opencrvs/
mosip.opencrvs.db.datasource.jdbc-url=jdbc:postgresql://192.168.1.67:5432/mosip_opencrvs
mosip.opencrvs.db.datasource.username=postgres
mosip.opencrvs.db.datasource.password=${db.dbuser.password}
mosip.opencrvs.db.datasource.birth.transaction.table=opencrvs.birth_transactions
mosip.opencrvs.db.datasource.cr.by=system
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
mosip.opencrvs.kafka.bootstrap.server=kafka-0.kafka-headless.mosip.svc.cluster.local:9092,kafka-1.kafka-headless.mosip.svc.cluster.local:9092,kafka-2.kafka-headless.mosip.svc.cluster.local:9092,kafka-3.kafka-headless.mosip.svc.cluster.local:9092
mosip.opencrvs.kafka.topic.name=OPENCRVS_BIRTH_RECORDS
mosip.opencrvs.kafka.topic.partitions=1
mosip.opencrvs.kafka.topic.replication.factor=1
mosip.opencrvs.kafka.admin.request.timeout.ms=2000
mosip.opencrvs.kafka.consumer.group.id=mediatorReceiver
mosip.opencrvs.kafka.consumer.poll.interval.ms=1000
mosip.opencrvs.kafka.consumer.auto.offset.reset=latest
mosip.opencrvs.kafka.consumer.enable.auto.commit=true
mosip.opencrvs.kafka.consumer.auto.commit.interval.ms=500
mosip.kernel.crypto.thumbprint.present=true
mosip.kernel.crypto.thumbprint.length=256
mosip.kernel.crypto.sign-algorithm-name=SHA512withRSA
mosip.opencrvs.auth.p12.filename=authkeystore.p12
mosip.opencrvs.cred.p12.filename=credkeystore.p12
mosip.opencrvs.p12.path=certs/
mosip.opencrvs.p12.password=${opencrvs.keystore.password}


#INIT Host configs
keymanager.msp.internal.api = kernel-keymanager-service.mosip.svc.cluster.local
syncdata.msp.internal.api = kernel-syncdata-service.mosip.svc.cluster.local
registrationstatus.msp.internal.api = dmz-regproc-registration-status-service.mosip.svc.cluster.local
packetreceiver.msp.internal.api = dmz-regproc-packet-receiver-stage.mosip.svc.cluster.local
ridgenerator.msp.internal.api = kernel-ridgenerator-service.mosip.svc.cluster.local
idrepository.identity.msp.internal.api = idrepo-identity-service.mosip.svc.cluster.local
idrepository.vid.msp.internal.api = idrepo-vid-service.mosip.svc.cluster.local

#Crypto Manager Apis
mosip.kernel.keymanager-service-auth-decrypt-url=http://${keymanager.msp.internal.api}/v1/keymanager/auth/decrypt
mosip.kernel.keymanager-service-CsSign-url=http://${keymanager.msp.internal.api}/v1/keymanager/cssign
mosip.kernel.keymanager-service-csverifysign-url=http://${keymanager.msp.internal.api}/v1/keymanager/csverifysign

CRYPTOMANAGER_ENCRYPT=http://${keymanager.msp.internal.api}/v1/keymanager/encrypt
CRYPTOMANAGER_DECRYPT=http://${keymanager.msp.internal.api}/v1/keymanager/decrypt

# Sync Data Apis and Regproc DMZ Apis
mosip.kernel.syncdata-service-get-tpm-publicKey-url=http://${keymanager.msp.internal.api}/v1/keymanager/tpmsigning/publickey
MIDSCHEMAURL=http://${syncdata.msp.internal.api}/v1/syncdata/latestidschema
SYNCSERVICE=http://${registrationstatus.msp.internal.api}/registrationprocessor/v1/registrationstatus/sync
PACKETRECEIVER=http://${packetreceiver.msp.internal.api}/registrationprocessor/v1/packetreceiver/registrationpackets
RIDGENERATION=http://${ridgenerator.msp.internal.api}/v1/ridgenerator/generate/rid
KEYMANAGER_TOKENID=http://kernel-keymanager-service.mosip.svc.cluster.local/v1/keymanager

#ID Repository Apis
IDREPO_IDENTITY=http://${idrepository.identity.msp.internal.api}/idrepository/v1/identity/
IDREPO_VID=http://${idrepository.vid.msp.internal.api}/idrepository/v1/vid
mosip.opencrvs.uin.token.intermediary.partner=mpartner-default-auth

```


---


# Step 5:  PMS create partner & policy 

***To enable credential Issue

***Create policy group : mpolicygroup-default-opencrvs
Create policy : mpolicy-default-opencrvs
Publish/Activate policy :     status (PUBLISHED)
Create partner : mpartner-default-opencrvs
Upload the partner certificate for Opencrvs mosip mediator to PMS
Upload Root CA , Intermediate CA , Partner certificate
Received signed certificate
Activate partner : status (approved)
map the credential type opencrvs to partner id by this API 
```URL
POST https://partnermanagerurl/v1/partnermanager/partners/{partnerId}/credentialType/opencrvs/policies/{policyName}
```

---



---


# Step 6: Openshift Deployment

***packet receiver stage
packet uploader stage
credential request generator
uin generator stage
opencrvs mediator
regproc opencrvs stage***