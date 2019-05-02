We have a multi-layered kustomize config laid out as follows (actually they're in different paths but this should illustrate the issue we have well enough):

```
.
└── dev1
    ├── dev
    │   ├── elasticstack-log
    │   │   ├── elasticstack
    │   │   │   ├── elasticsearch
    │   │   │   │   ├── elasticsearch-statefulSet.yml
    │   │   │   │   └── kustomization.yaml
    │   │   │   ├── elasticsearch-data
    │   │   │   │   ├── add-data-node-settings.yaml
    │   │   │   │   ├── elasticsearch-data-service.yml
    │   │   │   │   └── kustomization.yaml
    │   │   │   ├── elasticsearch-master
    │   │   │   │   ├── add-master-node-settings.yaml
    │   │   │   │   ├── elasticsearch-master-service.yml
    │   │   │   │   └── kustomization.yaml
    │   │   │   ├── kustomization.yaml
    │   │   │   ├── networkPolicy.yml
    │   │   │   └── README.md
    │   │   └── kustomization.yaml
    │   ├── kustomization.yaml
    │   └── properties
    │       └── cluster.properties
    ├── kustomization.yaml
    ├── properties
    │   └── cluster.properties
    └── README.md
```

We're trying to reduce the amount of double-keying as much as possible by having an elasticsearch StatefulSet in its own base and then deriving master and data nodes from it, but we've run into an issue when using variables. This is the base `kustomization.yaml`:

`dev1/dev/elasticstack-log/elasticstack/elasticsearch/kustomization.yaml`
```yaml
kind: Kustomization
apiVersion: kustomize.config.k8s.io/v1beta1

commonLabels:
  app.kubernetes.io/app: elasticsearch
  app: elasticsearch

resources:
  - elasticsearch-statefulSet.yml

images:
  - name: elasticsearch
    newTag: 6.6.0
```

And the base StatefulSet:

`dev1/dev/elasticstack-log/elasticstack/elasticsearch/elasticsearch-statefulSet.yml`
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
spec:
  serviceName: elasticsearch
  replicas: 3
  updateStrategy:
    type: RollingUpdate
  template:
    spec:
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
      priorityClassName: infra
      containers:
      - name: elasticsearch
        image: elasticsearch:6.6.0
        env:
          - name: CLUSTER_NAME
            value: logs
          - name: HTTP_ENABLE
            value: "true"
          - name: MINIMUM_NUMBER_OF_MASTERS
            value: "2"
          - name: NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: NETWORK_ADDRESS_CACHE_NEGATIVE_TTL
            value: "2"
          - name: NETWORK_ADDRESS_CACHE_TTL
            value: "2"
          - name: NETWORK_HOST
            value: "0.0.0.0"
          # PROCESSORS NEED TO BE AN INTEGER - SET LIMITS CPU ACCORDINGLY
          - name: PROCESSORS
            valueFrom:
              resourceFieldRef:
                resource: limits.cpu
        resources:
          requests:
            cpu: 0.25
          limits:
            cpu: 2
        ports:
        - containerPort: 9200
          name: http
        readinessProbe:
          httpGet:
            path: /_cluster/health?local=true
            port: 9200
          initialDelaySeconds: 5
        livenessProbe:
          tcpSocket:
            port: transport
          initialDelaySeconds: 40
          periodSeconds: 10
        volumeMounts:
        - name: storage
          mountPath: /data
```

And 