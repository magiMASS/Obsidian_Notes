# Maven Setup Guide – DEV & CLIENT Machines

## 1. Purpose of This Document

This document provides an **official, standardized guide** for setting up Maven on **DEV** and **CLIENT** machines to ensure:

·         Secure deployment of the **Core** project by the DEV team

·         Safe consumption of the Core dependency by the CLIENT team

·         **No credentials are exposed** to the client at any stage

·         Clear separation of responsibilities

---

## 2. Project Roles & Responsibilities

### 2.1 Core Project (Owned by DEV)

**Purpose**: Build and deploy shared libraries

**Maven Command**:

mvn clean deploy

**Responsibilities**: - Builds the Core artifact - Deploys artifacts to the internal repository - Manages repository credentials

---

### 2.2 Base Project (Owned by CLIENT)

**Purpose**: Consume Core as a dependency

**Maven Command**:

mvn clean install

**Responsibilities**: - Builds the Base project - Downloads Core artifacts from the repository - Never deploys artifacts

---

## 3. Repository Overview

All artifacts are hosted in the internal repository:

http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph

Repositories used: - **maven-public** – Dependency resolution (read-only) - **maven-snapshots** – Snapshot deployments (DEV only) - **maven-releases** – Release deployments (DEV only)

---

## 4. DEV Machine Setup (Core Project Owner)

### 4.1 Required Files

The DEV machine must have the following file:

~/.m2/settings.xml

⚠️ This file **must never be shared** with the client.

---

### 4.2 DEV – settings.xml (WITH credentials)

<settings>  <mirrors>  
    <mirror>  
      <id>nexus</id>  
      <mirrorOf>*,!central</mirrorOf>  
      <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-public/</url>  
    </mirror>  
  </mirrors>  
  
  <servers>  
    <server>  
      <id>nexus</id>  
      <username>admin</username>  
      <password>Admin@123</password>  
    </server>  
  </servers>  
  
</settings>
---

### 4.3 Core Project – pom.xml (REQUIRED)

The Core project **must declare** distributionManagement to enable deployment:

<distributionManagement>  <repository>  
    <id>nexus</id>  
    <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-releases</url>  
  </repository>  
  <snapshotRepository>  
    <id>nexus</id>  
    <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-snapshots</url>  
  </snapshotRepository>  
</distributionManagement>

This configuration enables:

mvn clean deploy

---

## 5. CLIENT Machine Setup (Base Project Owner)

### 5.1 Required Files

The CLIENT machine **does not require credentials**.

Optional file:

~/.m2/settings.xml

---

### 5.2 CLIENT – settings.xml (NO credentials)

<settings>  <mirrors>  
    <mirror>  
      <id>nexus</id>  
      <mirrorOf>*,!central</mirrorOf>  
      <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-public/</url>  
    </mirror>  
  </mirrors>  
  
</settings>

🚫 No <servers> section  
🚫 No username or password



### 5.3 Base Project – pom.xml (REQUIRED)

The CLIENT project **must declare a repository** for dependency resolution:

<repositories>  <repository>  
    <id>nexus</id>  
    <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-public/</url>  
    <releases>  
      <enabled>true</enabled>  
    </releases>  
    <snapshots>  
      <enabled>true</enabled>  
    </snapshots>  
  </repository>  
</repositories>

The Base project **must NOT** contain: - <distributionManagement> - <servers> - Any credentials

Client command:

mvn clean install

---

## 6. DEV vs CLIENT Comparison

|Aspect|DEV Machine|CLIENT Machine|
|---|---|---|
|Purpose|Build & deploy Core|Consume Core only|
|Maven command|mvn clean deploy|mvn clean install|
|settings.xml|With credentials|Without credentials|
|<servers>|✅ Present|❌ Absent|
|<distributionManagement>|✅ Required|❌ Not allowed|
|Repository access|Read + Write|Read-only|
|Credential exposure|Local only|None|

---

## 7. Final Summary

·         DEV machine deploys Core → credentials required

·         CLIENT machine builds Base → no credentials required

·         Core project defines deployment configuration

·         Base project defines repository for dependency resolution

This setup ensures **security, clarity, auditability, and seamless collaboration** between DEV and CLIENT teams.