We have a multi-layered kustomize config similar to the following (actually in practice they're in many different places to keep them modular but this should illustrate the issue we have):

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
    │       └── environment.properties
    ├── kustomization.yaml
    ├── properties
    │   └── cluster.properties
    └── README.md
```

We're trying to reduce the amount of double-keying as much as possible by having an elasticsearch StatefulSet in its own base and then deriving master and data nodes from it, but we've run into a naming issue. We want to name the `elasticsearch-master` and `elasticsearch-data` resources simply `elasticsearch` and then have the `nameSuffix` field append the necessary `-master` or `-data` strings onto the end, however when you run a `kustomize build dev1` it complains:

```
Error: Multiple matches for name ~G_v1_Service|monitoring|~P|elasticsearch|-master:-log:
  [~G_v1_Service|monitoring|~P|elasticsearch|-master:-log ~G_v1_Service|monitoring|~P|elasticsearch|-data:-log]
```

What are we doing wrong here? Are there any best practices for this kind of structure?
