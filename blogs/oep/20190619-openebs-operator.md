### Motivation
OpenEBS has various components that need to be in running state to let its volumes be consumed by higher applications. As the
number of components grow, have inter-dependency, need upgrades it becomes un-manageable to let this be managed by a human.
OpenEBS operator should ideally take over most of the manual operations needed to let OpenEBS run smoothly.

### Points to Consider
- Number of openebs components can vary
- Some of the openebs components can be optional
- Each component might need a special handling

### Possible Custom Resources
- Operator
- ReleaseConfig
- ReleaseJob
- ConfigInjector

### Sample Schemas

#### Operator - Sample 1
```yaml
kind: Operator
metadata:
  name: the-one-and-only
  namespace: openebs
spec:
  # installs or upgrades to OpenEBS 1.1.0
  version: 1.1.0
```

#### Operator - Sample 2
```yaml
kind: Operator
metadata:
  name: the-one-and-only
  namespace: openebs
spec:
  # installs or upgrades to OpenEBS 1.1.0
  version: 1.1.0
  components:
  # installs local provisioner only
  # un-installs other components if 
  # installed previously
  - name: LocalProvisioner
```

#### Operator - Sample 3
```yaml
kind: Operator
metadata:
  name: the-one-and-only
  namespace: openebs
spec:
  # installs or upgrades to OpenEBS 1.1.0
  version: 1.1.0  
  components:
  # installs specified components
  # un-installs other components if
  # installed previously
  - name: LocalProvisioner
  - name: MayaAPIServer
  - name: ExternalCSIProvisioner
  - name: NDM
```

#### Operator - Sample 4
```yaml
kind: Operator
metadata:
  name: the-one-and-only
  namespace: openebs
spec:
  # installs or upgrades to OpenEBS 1.1.0
  version: 1.1.0
  # optional config name; operator can
  # look up this config by using its 
  # metadata.name value
  config: the-one-and-only
```

#### Operator - Sample 5
```yaml
kind: Operator
metadata:
  name: the-one-and-only
  namespace: openebs
spec:
  # installs or upgrades to OpenEBS 1.1.0
  version: 1.1.0

  # optional release based jobs; operator
  # can look up this release job by using
  # its metadata.name value
  job: the-one-and-only
```

#### ReleaseConfig - Sample 1
```yaml
kind: ReleaseConfig
metadata:
  name: the-one-and-only
  namespace: openebs
spec:
  components:
  # these components are understood internally
  # i.e. controller logic via their names;
  # controller understands if a component is a 
  # deployment, or STS, or DaemonSet, or Job, etc.
  - name: MayaAPIServer
    image: quay.io/openebs/m-apiserver
    imageTag: 1.0.0
    namespace: openebs
    sidecars:
    - name: maya-exporter
      image: quay.io/openebs/m-exporter
      imageTag: 1.0.0
  - name: NDM
    image: quay.io/openebs/ndm
    imageTag: 0.5.0
    namespace: openebs
```

#### ReleaseJob - Sample 1
```yaml
kind: ReleaseJob
metadata:
  name: the-one-and-only
  namespace: openebs
spec:
  tasks:
  - name: DeleteOrphanCStorVolume
    image: quay.io/openebs/m-orhpancstordel
    imageTag: 0.0.1
    namespace: openebs
```

#### ConfigInjector - Sample 1
```yaml
```

#### References
- [OEP on CSI](https://github.com/openebs/openebs/pull/2617/)