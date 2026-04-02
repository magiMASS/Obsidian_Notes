# 🚀 Multiprotect + Java App Deployment on OpenShift (Sidecar Architecture)

---

## 📌 Overview

This project deploys a **Java Spring Boot application** alongside a **Multiprotect License Daemon** using a **sidecar container architecture** on OpenShift.

### 🧩 Architecture

```
[POD]
 ├── 🟢 license-daemon (Multiprotect_License_Daemon)
 └── 🔵 java-app (Spring Boot application)
```

- **Daemon container** → runs the license service
    
- **Java container** → waits for daemon readiness and starts the app
    

---

## 🛠️ Base OS

- Rocky Linux 8 (compatible with older CPUs and Multiprotect RPM)
    

---

## 📦 Docker Images

### 🟢 Daemon Image

- Installs Multiprotect RPM
    
- Copies license config
    
- Runs:
    
    ```
    /usr/sbin/Multiprotect/Multiprotect_License_Daemon
    ```
    

---

### 🔵 Java App Image

- Installs Java 11 (headless)
    
- Installs Multiprotect (for Manager CLI)
    
- Runs `start.sh`
    

---

## ⚙️ Startup Logic

### `start.sh`

The Java container waits until the daemon is ready:

```
while true:
  run Multiprotect_Manager
  if output contains "The Protection Service is not available":
      wait
  else:
      start Java app
```

---

## 🏗️ Build & Push Images

```bash
# Build daemon
docker build -t <repo>/multiprotect-daemon:latest -f Dockerfile.daemon .
docker push <repo>/multiprotect-daemon:latest

# Build java app
docker build -t <repo>/biosdk-app:latest -f Dockerfile.app .
docker push <repo>/biosdk-app:latest
```

---

## 🚀 OpenShift Deployment

### Apply YAML

```bash
oc apply -f deployment.yaml
oc apply -f service.yaml
oc apply -f route.yaml
```

---

## 🔍 Health Checks

### Readiness & Liveness Probe (Daemon)

```
/usr/sbin/Multiprotect/Multiprotect_Manager | grep -qv "The Protection Service is not available"
```

✔ Ensures daemon is fully operational

---

## 🌐 Service & Route

- Service exposes port `7788`
    
- Route provides external access
    

---

## 🧪 Verification

```bash
oc get pods
oc logs -f <pod> -c java-app
```

Expected output:

```
Waiting for License Daemon...
Daemon not ready...
Daemon READY ✅
Starting Java App...
```

---

## ⚠️ Common Issues & Fixes

|Issue|Solution|
|---|---|
|Multiprotect_Manager not found|Install RPM in Java container|
|JAR file missing|Fix path `/home/Biosdk/...`|
|Image pull error|Verify tag and push image|
|Permission denied|`chmod -R 777`|
|CrashLoopBackOff|Check container logs|

---

## 🧠 Key Design Decisions

- No `systemd` inside container
    
- No background (`&`) processes
    
- One process per container
    
- Sidecar pattern for separation of concerns
    
- Real readiness check (not sleep-based)
    

---

## ✅ Result

- Stable OpenShift deployment
    
- Proper lifecycle management
    
- Reliable startup synchronization
    
- Production-ready architecture
    

---

## 📈 Future Improvements

- Add HPA (autoscaling)
    
- Use ConfigMaps & Secrets
    
- Optimize image size
    
- Add monitoring & logging
    

---

## 👨‍💻 Author Notes

This setup is optimized for:

- Legacy RPM-based services (Multiprotect)
    
- OpenShift/Kubernetes environments
    
- CPU-constrained systems (no x86-64-v2 support)
    

---

## 🎯 Summary

✔ Sidecar architecture  
✔ OpenShift-native  
✔ Reliable startup handling  
✔ Compatible & production-ready

---