spec:
  replicas: 1
  selector:
    matchLabels:
      app: biosdk-msp
  template:
    metadata:
      labels:
        app: biosdk-msp
    spec:
      containers:
        - name: license-daemon
          image: docker.io/mspeagle/idemia-license:1.0
          resources: {}
          livenessProbe:
            exec:
              command:
                - sh
                - -c
                - >
                  /usr/sbin/Multiprotect/Multiprotect_Manager |
                  grep -qv "The Protection Service is not available"
            initialDelaySeconds: 60
            timeoutSeconds: 10
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 30
          readinessProbe:
            exec:
              command:
                - sh
                - -c
                - >
                  /usr/sbin/Multiprotect/Multiprotect_Manager |
                  grep -qv "The Protection Service is not available"
            initialDelaySeconds: 50
            timeoutSeconds: 10
            periodSeconds: 5
            successThreshold: 1
            failureThreshold: 30
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always

        - name: biosdk-app
          image: docker.io/mspeagle/biosdk-service-msp:01APR2026-1.0
          ports:
            - containerPort: 9099
              protocol: TCP
          env:
            - name: spring_config_url_env
              value: http://config-server.mosip.svc.cluster.local:80/config
            - name: spring_config_label_env
              value: "1.1.5.5"
            - name: active_profile_env
              value: mz
            - name: JDK_JAVA_OPTIONS
              value: "-Xms3000M -Xmx4000M"
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always

      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      imagePullSecrets:
        - name: mspeaglesecret
      schedulerName: default-scheduler

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%

  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600

status:
  observedGeneration: 16
  replicas: 1
  updatedReplicas: 1
  readyReplicas: 1
  availableReplicas: 1
  conditions:
    - type: Progressing
      status: "True"
      lastUpdateTime: "2026-04-02T09:16:32Z"
      lastTransitionTime: "2026-04-02T06:44:00Z"
      reason: NewReplicaSetAvailable
      message: ReplicaSet "biosdk-msp-75c8c96975" has successfully progressed.
    - type: Available
      status: "True"
      lastUpdateTime: "2026-04-02T10:17:39Z"
      lastTransitionTime: "2026-04-02T10:17:39Z"
      reason: MinimumReplicasAvailable
      message: Deployment has minimum availability.