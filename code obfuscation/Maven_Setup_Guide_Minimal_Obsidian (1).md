# 🚀 Maven Setup Guide – DEV & CLIENT Machines

> [!info]
> Standardized Maven setup for secure DEV deployment and CLIENT dependency usage.

---

## 📚 Table of Contents

- [Purpose](#-purpose)
- [Roles & Responsibilities](#-roles--responsibilities)
- [Repository Overview](#-repository-overview)
- [DEV Machine Setup](#-dev-machine-setup)
- [CLIENT Machine Setup](#-client-machine-setup)
- [DEV vs CLIENT](#-dev-vs-client)
- [Final Summary](#-final-summary)

---

# 🎯 Purpose

This guide ensures:

- Secure deployment of Core artifacts
- Safe dependency consumption by CLIENT projects
- No credential exposure
- Clear separation of responsibilities

---

# 👥 Roles & Responsibilities

## 🏗 Core Project (DEV)

### Command

```bash
mvn clean deploy
```

### Responsibilities

- Build Core artifact
- Deploy artifacts
- Manage credentials

---

## 📦 Base Project (CLIENT)

### Command

```bash
mvn clean install
```

### Responsibilities

- Build Base project
- Download dependencies
- Never deploy artifacts

---

# 🌐 Repository Overview

```text
http://artifactory-http-mosip.apps.uat2.phylsys.gov.ph
```

| Repository | Purpose |
|---|---|
| `maven-public` | Dependency resolution |
| `maven-snapshots` | Snapshot deployments |
| `maven-releases` | Release deployments |

---

# 🛠 DEV Machine Setup

> [!warning]
> Never share DEV credentials with CLIENT teams.

## Required File

```text
~/.m2/settings.xml
```

## DEV `settings.xml`

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

## Core `pom.xml`

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

---

# 👨‍💻 CLIENT Machine Setup

> [!tip]
> CLIENT machines only require read access.

## Optional File

```text
~/.m2/settings.xml
```

## CLIENT `settings.xml`

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

## Base `pom.xml`

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

> [!danger]
> CLIENT projects must NOT contain:
>
> - `<distributionManagement>`
> - `<servers>`
> - Credentials

---

# ⚖️ DEV vs CLIENT

| Feature | DEV | CLIENT |
|---|---|---|
| Command | `mvn clean deploy` | `mvn clean install` |
| Credentials | ✅ | ❌ |
| `<servers>` | ✅ | ❌ |
| Deployment | ✅ | ❌ |
| Access | Read + Write | Read-only |

---

# ✅ Final Summary

- DEV machines deploy Core artifacts
- CLIENT machines consume dependencies only
- Credentials stay local to DEV machines
- Repository access remains secure


ok