```
[root@ip-10-10-6-201 prometheus-kubernetes]# cat node-exporter-ds.yaml node-exporter-svc.yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    app: node-exporter
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
      name: node-exporter
    spec:
      containers:
      - args:
        - -collector.procfs=/host/proc
        - -collector.sysfs=/host/sys
        image: 54.223.110.70/kubernetes/node-exporter:v0.14.0
        imagePullPolicy: IfNotPresent
        name: node-exporter
        ports:
        - containerPort: 9100
          hostPort: 9100
          name: scrape
          protocol: TCP
        resources:
          limits:
            cpu: 200m
            memory: 50Mi
          requests:
            cpu: 100m
            memory: 30Mi
        volumeMounts:
        - mountPath: /host/proc
          name: proc
          readOnly: true
        - mountPath: /host/sys
          name: sys
          readOnly: true
      dnsPolicy: ClusterFirst
      hostNetwork: true
      hostPID: true
      restartPolicy: Always
      schedulerName: default-scheduler
      volumes:
      - hostPath:
          path: /proc
        name: proc
      - hostPath:
          path: /sys
        name: sys
  updateStrategy:
    type: OnDelete
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: node-exporter
    k8s-app: node-exporter
  name: node-exporter
  namespace: monitoring
spec:
  clusterIP: None
  ports:
  - name: http-metrics
    port: 9100
    protocol: TCP
    targetPort: 9100
  selector:
    app: node-exporter
  sessionAffinity: None
  type: ClusterIP

```



