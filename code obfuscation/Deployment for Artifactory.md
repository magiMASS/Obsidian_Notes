# Nexus3 Deployment Guide on Red Hat OpenShift

## Overview

This document explains how to deploy Nexus Repository Manager (`sonatype/nexus3:3.88.0`) on Red Hat OpenShift using:

- Deployment
- Persistent Volume Claim (PVC)
- Service
- OpenShift Route

The deployment provides persistent storage and external access to Nexus Repository Manager.

---

# Architecture

| Component | Description |
|---|---|
| Deployment | Manages Nexus pod lifecycle |
| Pod | Runs Nexus Repository Manager |
| PVC (`nexus-data`) | Persistent storage for Nexus data |
| Service | Internal cluster access |
| Route | External OpenShift access |
| Namespace/Project | `mosip` |

---

# Prerequisites

Ensure the following before deployment:

- OpenShift cluster access
- `oc` CLI installed
- User has deployment permissions
- Storage class available in cluster

---

# Step 1: Login to OpenShift
# Step 2 : Create PersistentVolumeClaim


```
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: nexus-data
  namespace: mosip

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 20Gi
```











;