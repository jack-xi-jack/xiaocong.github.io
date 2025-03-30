---
title: 部署Prometheus监控
date: 2025-03-11 18:00:00
---
> <font style="background-color:rgba(255, 255, 255, 0);">如果已安装metrics-server需要先卸载，否则冲突</font>
>

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<h1 id="adfb3ce8"><font style="background-color:rgba(255, 255, 255, 0);">组件说明</font></h1>
---

1. <font style="background-color:rgba(255, 255, 255, 0);">MetricServer：是kubernetes集群资源使用情况的聚合器，收集数据给kubernetes集群内使用，如kubectl,hpa,scheduler等。</font>
2. <font style="background-color:rgba(255, 255, 255, 0);">PrometheusOperator：是一个系统监测和警报工具箱，用来存储监控数据。</font>
3. <font style="background-color:rgba(255, 255, 255, 0);">NodeExporter：用于各node的关键度量指标状态数据。</font>
4. <font style="background-color:rgba(255, 255, 255, 0);">KubeStateMetrics：收集kubernetes集群内资源对象数据，制定告警规则。</font>
5. <font style="background-color:rgba(255, 255, 255, 0);">Prometheus：采用pull方式收集apiserver，scheduler，controller-manager，kubelet组件数据，通过http协议传输。</font>
6. <font style="background-color:rgba(255, 255, 255, 0);">Grafana：是可视化数据统计和监控平台。</font>

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<h1 id="039d392c"><font style="background-color:rgba(255, 255, 255, 0);">安装部署</font></h1>
---

<font style="background-color:rgba(255, 255, 255, 0);">项目地址：</font>[<font style="background-color:rgba(255, 255, 255, 0);">https://github.com/prometheus-operator/kube-prometheus</font>](https://github.com/prometheus-operator/kube-prometheus)

<h2 id="6a6cc2b2"><font style="background-color:rgba(255, 255, 255, 0);">克隆项目至本地</font></h2>
---

```plain
git clone https://github.com/prometheus-operator/kube-prometheus.git
```

<h2 id="1dcabd41"><font style="background-color:rgba(255, 255, 255, 0);">创建资源对象</font></h2>
---

```plain
[root@master1 k8s-install]# kubectl create namespace monitoring 
[root@master1 k8s-install]# cd kube-prometheus/
[root@master1 kube-prometheus]# kubectl apply --server-side -f manifests/setup
[root@master1 kube-prometheus]# kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
[root@master1 kube-prometheus]# kubectl apply -f manifests/
```

<h2 id="fea33636"><font style="background-color:rgba(255, 255, 255, 0);">验证查看</font></h2>
---

+ <font style="background-color:rgba(255, 255, 255, 0);">查看pod状态</font>

```plain
[root@master1 kube-prometheus]# kubectl get pod -n monitoring 
NAME                                   READY   STATUS    RESTARTS   AGE
alertmanager-main-0                    2/2     Running   0          61s
alertmanager-main-1                    2/2     Running   0          61s
alertmanager-main-2                    2/2     Running   0          61s
blackbox-exporter-576df9484f-lr6xd     3/3     Running   0          107s
grafana-795ddfd4bd-jxlrw               1/1     Running   0          105s
kube-state-metrics-bdfdcd5cd-7dgwl     3/3     Running   0          104s
node-exporter-4qnrz                    2/2     Running   0          104s
node-exporter-8hjr7                    2/2     Running   0          104s
node-exporter-8s5hp                    2/2     Running   0          103s
node-exporter-kgb48                    2/2     Running   0          104s
node-exporter-p8b7q                    2/2     Running   0          103s
node-exporter-v4nz7                    2/2     Running   0          103s
prometheus-adapter-65b6bd474c-qvdb8    1/1     Running   0          102s
prometheus-adapter-65b6bd474c-vlxhn    1/1     Running   0          102s
prometheus-k8s-0                       1/2     Running   0          58s
prometheus-k8s-1                       2/2     Running   0          58s
prometheus-operator-6565b7b5f5-mgclf   2/2     Running   0          101s
```

+ <font style="background-color:rgba(255, 255, 255, 0);">查看top信息</font>

```plain
[root@master1 kube-prometheus]# kubectl top node
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master1   579m         14%    2357Mi          66%       
master2   383m         9%     1697Mi          48%       
master3   482m         12%    2069Mi          58%       
work1     131m         3%     1327Mi          37%       
work2     132m         3%     1134Mi          32%       
work3     176m         4%     1100Mi          31%       
[root@master1 kube-prometheus]# kubectl top pod
NAME                     CPU(cores)   MEMORY(bytes)   
myapp-58bbc79c4f-cc9g5   0m           1Mi             
myapp-58bbc79c4f-txnp5   0m           1Mi             
myapp-58bbc79c4f-zvlcr   0m           1Mi
```

<h2 id="200078c4"><font style="background-color:rgba(255, 255, 255, 0);">新增ingress资源</font></h2>
---

<font style="background-color:rgba(255, 255, 255, 0);">以ingress-nginx为例：</font>

```plain
[root@master1 manifests]# cat ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alertmanager
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: alertmanager.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: alertmanager-main
            port:
              number: 9093 
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: prometheus.local.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-k8s
            port:
              number: 9090
```

<h2 id="10b4cf9a"><font style="background-color:rgba(255, 255, 255, 0);">web访问验证</font></h2>
---

+ <font style="background-color:rgba(255, 255, 255, 0);">新增hosts解析记录</font>

```plain
192.168.10.10 alertmanager.local.com grafana.local.com prometheus.local.com
```

+ <font style="background-color:rgba(255, 255, 255, 0);">访问</font>[<font style="background-color:rgba(255, 255, 255, 0);">http://alertmanager.local.com/</font>](http://alertmanager.local.com/)<font style="background-color:rgba(255, 255, 255, 0);">，查看当前激活的告警</font>

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1737813858882-40dfbb97-749d-4cf4-b79b-02dfa018a3fa.jpeg)

+ <font style="background-color:rgba(255, 255, 255, 0);">访问</font>[<font style="background-color:rgba(255, 255, 255, 0);">http://prometheus.local.com/targets</font>](http://prometheus.local.com/targets)<font style="background-color:rgba(255, 255, 255, 0);">，查看targets已全部up</font>

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1737813870278-f52a4a68-aca7-4ea7-8964-0be9efccd42e.jpeg)

+ <font style="background-color:rgba(255, 255, 255, 0);">访问</font>[<font style="background-color:rgba(255, 255, 255, 0);">http://grafana.local.com/login</font>](http://grafana.local.com/login)<font style="background-color:rgba(255, 255, 255, 0);">，默认用户名和密码是admin/admin</font>

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1737813882935-61131e36-ed0a-41fd-9c20-ea4e8a71edc2.jpeg)

+ <font style="background-color:rgba(255, 255, 255, 0);">查看数据源，以为我们自动配置Prometheus数据源</font>

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1737813895510-86a615e8-c0a0-4c36-bb6b-b9b40d22ed5d.jpeg)

<h2 id="8094d557"><font style="background-color:rgba(255, 255, 255, 0);">targets异常处理</font></h2>
---

<font style="background-color:rgba(255, 255, 255, 0);">查看targets可发现有两个监控任务没有对应的instance，这和serviceMonitor资源对象有关</font>

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1737813909963-bdc64819-89b3-4ed4-9071-a8e3fcd6c7e6.jpeg)

<font style="background-color:rgba(255, 255, 255, 0);">由于prometheus-serviceMonitorKubeScheduler文件中，selector匹配的是service的标签，但是namespace中并没有app.kubernetes.io/name的service</font>

1. <font style="background-color:rgba(255, 255, 255, 0);">新建prometheus-kubeSchedulerService.yaml并apply创建资源</font>

```plain
apiVersion: v1
kind: Service
metadata:
    namespace: kube-system
    name: kube-scheduler
    labels:
      app.kubernetes.io/name: kube-scheduler
spec:
    selector:
      component: kube-scheduler
    type: ClusterIP
    ports:
    - name: https-metrics
      port: 10259
      targetPort: 10259
      protocol: TCP
```

2. <font style="background-color:rgba(255, 255, 255, 0);">新建prometheus-kubeControllerManagerService.yaml并apply创建资源</font>

```plain
apiVersion: v1
kind: Service
metadata:
    namespace: kube-system
    name: kube-controller-manager
    labels:
      app.kubernetes.io/name: kube-controller-manager
spec:
    selector:
      component: kube-controller-manager
    type: ClusterIP
    ports:
    - name: https-metrics
      port: 10257
      targetPort: 10257
      protocol: TCP
```

3. <font style="background-color:rgba(255, 255, 255, 0);">再次查看targets信息</font>

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1737813969703-36af3983-6b25-42e6-afaf-b85b13f7855e.jpeg)

<font style="background-color:rgba(255, 255, 255, 0);">发现虽然加载了targets，但是无法访问该端口。</font>

<font style="background-color:rgba(255, 255, 255, 0);">需要请修改master节点的/etc/kubernetes/manifests/kube-controller-manager.yaml 文件和 /etc/kubernetes/manifests/kube-scheduler.yaml 文件，将其中的 - --bind-address=127.0.0.1 修改为 - --bind-address=0.0.0.0</font>

<font style="background-color:rgba(255, 255, 255, 0);">修改完保存文件，pod会自动重启。</font>

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<h1 id="48a6122b"><font style="background-color:rgba(255, 255, 255, 0);">部署pushgateway(可选)</font></h1>
---

<h2 id="5e69a94c"><font style="background-color:rgba(255, 255, 255, 0);">创建资源清单</font></h2>
---

<font style="background-color:rgba(255, 255, 255, 0);">pushgateway目录下，创建这三个yaml文件。</font>

+ <font style="background-color:rgba(255, 255, 255, 0);">prometheus-pushgatewayServiceMonitor.yaml</font>

```plain
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    prometheus: k8s
  name: prometheus-pushgateway
  namespace: monitoring
spec:
  endpoints:
  - honorLabels: true
    port: http
  jobLabel: pushgateway
  selector:
    matchLabels:
      app: prometheus-pushgateway
```

+ <font style="background-color:rgba(255, 255, 255, 0);">prometheus-pushgatewayService.yaml</font>

```plain
apiVersion: v1
kind: Service
metadata:
  labels:
    app: prometheus-pushgateway
  name: prometheus-pushgateway
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: http
    port: 9091
    nodePort: 30400
    targetPort: metrics
  selector:
    app: prometheus-pushgateway
#  type: ClusterIP
```

+ <font style="background-color:rgba(255, 255, 255, 0);">prometheus-pushgatewayDeployment.yaml</font>

```plain
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: prometheus-pushgateway
  name: prometheus-pushgateway
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-pushgateway
  template:
    metadata:
      labels:
        app: prometheus-pushgateway
    spec:
      containers:
      - image: prom/pushgateway:v1.10.0
        livenessProbe:
          httpGet:
            path: /#/status
            port: 9091
          initialDelaySeconds: 10
          timeoutSeconds: 10
        name: prometheus-pushgateway
        ports:
        - containerPort: 9091
          name: metrics
        readinessProbe:
          httpGet:
            path: /#/status
            port: 9091
          initialDelaySeconds: 10
          timeoutSeconds: 10
        resources:
          limits:
            cpu: 50m
            memory: 100Mi
          requests:
            cpu: 50m
            memory: 100Mi
```

+ <font style="background-color:rgba(255, 255, 255, 0);">prometheus-ingress.yaml</font>

```plain
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: prometheus-pushgateway
  namespace: monitoring
spec:
  entryPoints:
  - web
  routes:
  - match: Host(`prometheus-pushgateway.local.com`)
    kind: Rule
    services:
      - name: prometheus-pushgateway
        port: 9091
```

<h2 id="e8928169"><font style="background-color:rgba(255, 255, 255, 0);">创建资源</font></h2>
---

```plain
kubectl apply -f .
```

<h2 id="eca8abf0"><font style="background-color:rgba(255, 255, 255, 0);">查看验证</font></h2>
---

![](https://cdn.nlark.com/yuque/0/2025/jpeg/43141749/1737814086186-b0c70632-a294-474f-8d17-9514ff77cd90.jpeg)

<h1 id="1f318234"><font style="background-color:rgba(255, 255, 255, 0);">高级配置</font></h1>
---

<h2 id="3b5e4f43"><font style="background-color:rgba(255, 255, 255, 0);">Grafana配置修改</font></h2>
---

<font style="background-color:rgba(255, 255, 255, 0);">默认grafana使用UTC时区和sqllite数据库，可按需调整</font>

```plain
# pwd
/opt/k8s/kube-prometheus/manifests
# cat grafana-config.yaml                
apiVersion: v1
kind: Secret
metadata:
  labels:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 11.2.1
  name: grafana-config
  namespace: monitoring
stringData:
  grafana.ini: |
    [date_formats] # 时区设置
    default_timezone = Asia/Shanghai
    [database] # 数据库设置
    type = mysql
    host = cluster-mysql-master.mysql.svc:3306
    name = grafana
    user = grafana
    password = password
type: Opaque
```

<h2 id="e80a0cad"><font style="background-color:rgba(255, 255, 255, 0);">数据持久化存储</font></h2>
---

<font style="background-color:rgba(255, 255, 255, 0);">默认的的存储为</font>`<font style="background-color:rgba(255, 255, 255, 0);">emptyDir</font>`<font style="background-color:rgba(255, 255, 255, 0);">，生产环境建议更换为</font>`<font style="background-color:rgba(255, 255, 255, 0);">persistentVolumeClaim</font>`

<font style="background-color:rgba(255, 255, 255, 0);">创建grafana-pvc</font>

```plain
# cat grafana-pvc.yaml      
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: grafana-pvc
  namespace: monitoring
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
# kubectl apply -f grafana-pvc.yaml   
persistentvolumeclaim/grafana-pvc created
```

<font style="background-color:rgba(255, 255, 255, 0);">修改grafana</font>

```plain
# cp /opt/k8s/kube-prometheus/manifests/grafana-deployment.yaml .
# vim grafana-deployment.yaml 
volumes:
- name: grafana-storage
  persistentVolumeClaim:
    claimName: grafana-pvc
# kubectl apply -f grafana-deployment.yaml 
deployment.apps/grafana configured
```

<font style="background-color:rgba(255, 255, 255, 0);">修改prometheus</font>

```plain
# cp /opt/k8s/kube-prometheus/manifests/prometheus-prometheus.yaml .
# vim prometheus-prometheus.yaml 
spec:
  image: quay.io/prometheus/prometheus:v2.54.1
  retention: 30d # 数据保留天数
  storage: # 持久化配置
    volumeClaimTemplate:
      spec:
        storageClassName: nfs   
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 500Gi
# kubectl apply -f prometheus-prometheus.yaml          
prometheus.monitoring.coreos.com/k8s configured
```

<h2 id="f28d5761"><font style="background-color:rgba(255, 255, 255, 0);">node exporter新增ip标签</font></h2>
---

<font style="background-color:rgba(255, 255, 255, 0);">默认情况下node exporter指标只有主机名没有ip标签，可添加全局IP标签。</font>

```plain
# cp ../kube-prometheus/manifests/nodeExporter-serviceMonitor.yaml .
# vim nodeExporter-serviceMonitor.yaml 
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/component: exporter
    app.kubernetes.io/name: node-exporter
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 1.8.2
  name: node-exporter
  namespace: monitoring
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    interval: 15s
    port: https
    relabelings:
    - action: replace
      regex: (.*)
      replacement: $1
      sourceLabels:
      - __meta_kubernetes_pod_node_name
      targetLabel: instance
    - action: replace
      sourceLabels: 
      - __meta_kubernetes_pod_host_ip
      targetLabel: ip
    scheme: https
    tlsConfig:
      insecureSkipVerify: true
  jobLabel: app.kubernetes.io/name
  selector:
    matchLabels:
      app.kubernetes.io/component: exporter
      app.kubernetes.io/name: node-exporter
      app.kubernetes.io/part-of: kube-prometheus
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

