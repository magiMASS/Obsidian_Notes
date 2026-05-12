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
# Step 1 : Create PersistentVolumeClaim


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




# Step 3: Create Nexus Deployment
```
apiVersion: apps/v1
kind: Deployment

metadata:
  name: artifactory
  namespace: mosip

spec:
  replicas: 1

  selector:
    matchLabels:
      app: artifactory

  template:
    metadata:
      labels:
        app: artifactory

    spec:
      securityContext:
        fsGroup: 200

      volumes:
        - name: nexus-data
          persistentVolumeClaim:
            claimName: nexus-data

      containers:
        - name: artifactory
          image: sonatype/nexus3:3.88.0

          ports:
            - containerPort: 8081
              protocol: TCP

          volumeMounts:
            - name: nexus-data
              mountPath: /nexus-data

          imagePullPolicy: Always

      restartPolicy: Always

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%

  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```


# Step 6: Create Service

```
apiVersion: v1
kind: Service

metadata:
  name: artifactory-service
  namespace: mosip

spec:
  selector:
    app: artifactory

  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081

  type: ClusterIP
```



