### Specification
- All the things that are shown here will not be supported in the beginning
- However, it is important to get the naming & design spec right by considering wide number of use cases

```yaml
# desired state
spec:
  # refers to openebs components that should be available & in the specified state
  component:

  # simple desired state specification
  - include: currentCASTemplate; currentRunTask; currentOpenEBSProvisioner
  - exclude: labelSelector app eq jiva version lt current

  # finely granular desired state specification
  - exclude: selector kind eq runtask
    if: selector name contains -0.6.0
  - include: selector kind eq castemplate
    replace: metadata.labels.abc
    value: |
    if: labelSelector version eq 0.6.0; pathSelector metadata.labels.abc ne default

  # refers to features supported by openebs and should work as specified here
  feaure:
  - support: CreateJivaVolume; NVMEIsAvailable; HostHasISCSIInstalled
  - support: CreateCstorVolume
    if: selector openebsVersion gt 0.7.0
  - support: VolumeOnSparseFiles
    if: labelSelector stage in test,qa
  - support: CreateCstorVolume
    check: latency lt 1msec
    with: replicas 3
    interval: 1day or 1 hr # check support every 1 day on success or every 1 hr on failure

status:
  state: failed
  reason:
  starttime:
  endtime:
  feature:
  - support: CreateJivaVolume; NVMEIsAvailable
    state: passed
  - support: CreateCstorVolume
    if: selector version gt 0.7.0
    state: passed
  - support: CreateCstorVolume
    check: latency lt 1msec
    with: replicas 3
    interval: 3days
    state: passed
    starttime:
    endtime:
    prevState: failed
  component:
  - include: currentCASTemplate, currentRunTask, currentOpenEBSProvisioner
    state: passed
  - exclude: labelSelector app eq jiva version lt current
    state: passed
  - exclude: selector kind eq runtask
    if: selector name contains -0.6.0
    state: passed
  - include: selector kind eq castemplate
    replace: metadata.labels.abc
    value: |
    if: labelSelector version eq 0.6.0, pathSelector metadata.labels.abc ne default
    state: failed
    reason: blah blah blah
    starttime:
    endtime:
    count:
```