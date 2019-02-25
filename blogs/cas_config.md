### Info
- Version: 2
- Last Updated On: 25 Feb 2019

### Motivation
Desire to apply, inject, merge configuration against components or services in a kubernetes cluster.

### High Level Design
- CASConfig is a kubernetes _custom resource_
- It can be **optionally** controlled by its own controller
  - Name of its controller is called as "cas-config controller"
- It can be embedded inside other resources, e.g.:
  - OpenebsCluster
  - CstorVolume (_Future_)
  - CstorPoolClaim (_Future_)
  - JivaVolume (_Future_)
- The embedding object should embed only a single config specification
- A config can have one or more targets
  - A config target is selected via `spec.select` option
  - A config target is one on which config is applied i.e. injected
- It can patch, merge, add following configurations:
  - labels
  - annotations
  - envs
  - containers
  - sidecars
  - maincontainer
  - tolerations
  - etc

### Specifications of CASConfig
```go
type CASConfig struct {
  metav1.TypeMeta
  metav1.ObjectMeta
  
  Spec    Spec    `json:"spec"`
  Status  Status  `json:"status"`
}

type Spec struct {
  Scope   Scope     `json:"scope"`
  Groups  []Group   `json:"groups"`
}

type ScopeType

const (
  ClusterScope    ScopeType = "cluster"
  NamespaceScope  ScopeType = "namespace"
)

type Scope struct {
  Level   ScopeType   `json:"level"`
}

type Group struct {
  Name      string      `json:"name"`
  Values    Values      `json:"values"`
  Selector  Selector    `json:"select"`
}

type Values struct {
  Labels      map[string]string `json:"labels"`
  Annotations map[string]string `json:"annotations"`
}

type ValueOps struct {
  LabelOps    []LabelOperation  `json:"labels"`
}

type LabelOperation struct {
  Kind      string    `json:"kind"`
  Name      string    `json:"name"`
  Selector  Selector  `json:"select"`
  Path      string    `json:"path"`
}

type Selector struct {
  ByLabels        map[string]string `json:"byLabels"`
  ByLabelops      []LabelOperation  `json:"byLabelOps"`
  ByAnnotations   map[string]string `json:"byAnnotations"`
  ByNamespace     string            `json:"byNamespace"`
  ByKind          string            `json:"byKind"`
  ByName          string            `json:"byName"`
}
```

```yaml
kind: CASConfig
spec:
  scope:
    level:
    values:
  group:
  - name:
    values:
      labels:
      annotations:
      envs:
      containers:
      sidecars:
      maincontainer:
    valueOps:
      labels:
      - kind:
        name:
        select:
        path:
    select:
      byLabels:
      byLabelOps:
      byAnnotations:
      byNamespace:
      byKind:
      byName:
```