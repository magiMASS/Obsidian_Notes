---
tags:
  - maven
  - devops
  - artifactory
  - deployment
  - obsidian
created: 2026-05-11
---

# 🚀 Maven Setup Guide – DEV & CLIENT Machines

> [!info]
> Official standardized Maven setup guide for DEV and CLIENT environments.

---

# 📚 Table of Contents

- [[#🎯 Purpose of This Document]]
- [[#👥 Project Roles & Responsibilities]]
- [[#🌐 Repository Overview]]
- [[#🛠 DEV Machine Setup]]
- [[#👨‍💻 CLIENT Machine Setup]]
- [[#⚖️ DEV vs CLIENT Comparison]]
- [[#✅ Final Summary]]

---

# 🎯 Purpose of This Document

This document provides a secure and standardized Maven setup process to ensure:

- 🔐 Secure deployment of Core artifacts
- 📦 Safe dependency consumption by CLIENT projects
- 🚫 No credential exposure
- 🤝 Clear separation of responsibilities
- 🧩 Consistent repository configuration

> [!tip]
> This setup is designed for enterprise-grade collaboration between DEV and CLIENT teams.

---

# 👥 Project Roles & Responsibilities

## 🏗 Core Project (Owned by DEV)

> [!info]
> The Core project is responsible for building and deploying shared libraries.

### ⚡ Maven Command

```bash
mvn clean deploy
```

### ✅ Responsibilities

- Build the Core artifact
- Deploy artifacts to Artifactory
- Manage deployment credentials
- Publish snapshots and releases

---

## 📦 Base Project (Owned by CLIENT)

> [!info]
> The Base project consumes Core dependencies but never deploys artifacts.

### ⚡ Maven Command

```bash
mvn clean install
```

### ✅ Responsibilities

- Build the Base project
- Download dependencies from repository
- Consume Core libraries safely
- Avoid deployment operations

---

# 🌐 Repository Overview

> [!info]
> All artifacts are hosted in the internal Artifactory repository.

## 🔗 Repository URL

```text
http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph
```

---

## 📁 Repository Usage

| Repository | Purpose | Access |
|---|---|---|
| `maven-public` | Dependency resolution | 🔓 Read-only |
| `maven-snapshots` | Snapshot deployments | 🔐 DEV only |
| `maven-releases` | Release deployments | 🔐 DEV only |

---

# 🛠 DEV Machine Setup

> [!danger]
> DEV credentials must NEVER be shared with CLIENT teams.

---

## 📂 Required File

```text
~/.m2/settings.xml
```

> [!warning]
> This file contains deployment credentials and must remain local to DEV machines.

---

## ⚙️ DEV `settings.xml`

<details>
<summary>📄 Click to expand DEV settings.xml</summary>

```xml
<settings>

  <mirrors>
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
```

</details>

---

## 📄 Core Project `pom.xml`

> [!info]
> `distributionManagement` is REQUIRED for artifact deployment.

<details>
<summary>📄 Click to expand distributionManagement configuration</summary>

```xml
<distributionManagement>
  <repository>
    <id>nexus</id>
    <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-releases</url>
  </repository>

  <snapshotRepository>
    <id>nexus</id>
    <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-snapshots</url>
  </snapshotRepository>
</distributionManagement>
```

</details>

---

## 🚀 Deployment Command

```bash
mvn clean deploy
```

> [!success]
> This command builds and publishes artifacts to Artifactory.

---

# 👨‍💻 CLIENT Machine Setup

> [!tip]
> CLIENT machines only require read access to repositories.

---

## 📂 Optional File

```text
~/.m2/settings.xml
```

> [!info]
> CLIENT configuration does NOT require credentials.

---

## ⚙️ CLIENT `settings.xml`

<details>
<summary>📄 Click to expand CLIENT settings.xml</summary>

```xml
<settings>

  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*,!central</mirrorOf>
      <url>http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph/repository/maven-public/</url>
    </mirror>
  </mirrors>

</settings>
```

</details>

---

## 🚫 Security Restrictions

> [!danger]
> CLIENT machines must NEVER contain deployment credentials.

### ❌ Do NOT include:

- `<servers>`
- Username/password
- Deployment credentials
- `<distributionManagement>`

---

## 📄 Base Project `pom.xml`

> [!info]
> CLIENT projects only require repository definitions for dependency resolution.

<details>
<summary>📄 Click to expand repository configuration</summary>

```xml
<repositories>
  <repository>
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
```

</details>

---

## ⚡ CLIENT Build Command

```bash
mvn clean install
```

> [!success]
> Downloads dependencies and builds the project locally.

---

# ⚖️ DEV vs CLIENT Comparison

| Feature | 🛠 DEV Machine | 👨‍💻 CLIENT Machine |
|---|---|---|
| Purpose | Build & Deploy Core | Consume Core |
| Maven Command | `mvn clean deploy` | `mvn clean install` |
| Credentials | ✅ Required | ❌ Not Required |
| `<servers>` | ✅ Present | ❌ Absent |
| `<distributionManagement>` | ✅ Required | ❌ Not Allowed |
| Repository Access | 🔐 Read + Write | 🔓 Read-only |
| Artifact Deployment | ✅ Allowed | ❌ Not Allowed |
| Security Risk | Medium | Low |

---

# 🔐 Security Best Practices

> [!warning]
> Always follow these security rules.

## ✅ Recommended

- Keep credentials local to DEV machines
- Use repository mirrors
- Restrict deployment access
- Audit repository permissions regularly

---

## 🚫 Never Do This

- Share `settings.xml`
- Commit credentials to Git
- Add passwords inside `pom.xml`
- Allow CLIENT deployments

---

# ✅ Final Summary

> [!success]
> This architecture ensures secure, maintainable, and enterprise-ready Maven collaboration.

## 📌 Key Takeaways

- 🛠 DEV machines deploy artifacts
- 👨‍💻 CLIENT machines only consume dependencies
- 🔐 Credentials remain private
- 📦 Repository configuration stays centralized
- 🤝 Teams remain clearly separated

---

# 🧩 Related Notes

- [[Maven Repository Setup]]
- [[Artifactory Access Rules]]
- [[Core Project Deployment]]
- [[Maven Security Guidelines]]

---

> [!quote]
> “Good infrastructure documentation reduces deployment mistakes before they happen.”
