### Info
- Version: 4
- Last Updated On: 27 Feb 2019

### Motivation
What if there is an easy way to code adhoc stuff or automations with respect to Kubernetes whithout messing with shell scripts. This will be used internally as a handy tool & is not meant for production purposes. One such example can be writing this to achieve a test case automation.

### High Level Design - _MayaScript_
- It is a kubernetes custom resource
- It can be optionally managed by a controller
  - Name of its optional controller is "maya-script controller"
- It understands invoking kubernetes API calls
- It understands invoking openebs/maya functions
- It executes itself as a go template
- It can save the success result of each line of script code
- It can save the error result (if any) of the line of script code
- It can save the time taken to execute each line of script code
- It exits execution if any line of script code results in an error
- It can accept one or more yaml document(s)
- It should execute based on the order provided in the specifications

### _Specification_ of MayaScript
```go
// pkg/apis/openebs.io/mayascript/v1alpha1/mayascript.go

type MayaScript struct {
  metav1.TyeMeta
  metav1.ObjectMeta
  Spec   MayaScriptSpec
  Status MayaScriptStatus
}

type MayaScriptSpec struct {
  Script map[string]string `json:"script"`
}

type MayaScriptStatus struct {}
```

```go
// pkg/unstruct/v1alpha1/unstruct.go

type buildOption func(*unstruct)

func WithKind(kind string) buildOption {
  // ...
}

// Expose this as a go template function 
// with the name kubeKind
func WithKindOpt(kind string, opts ...buildOption) []buildOption {
  opts = append(opts, WithKind(kind))
  return opts
}
```

```yaml
kind: MayaScript
spec:
  title: I would like to manage my kubernetes scripts in a better way
  runs:
  - desc: I have a kubernetes cluster with 3 nodes
    withSCYaml: |
    withClusterRoleYaml: |
    withAPIServerYaml: |
    withNDMYaml: |
    withProvisionerYaml: |
  - it: should install openebs without problems
    installSC: $(kubeApply .yaml.withSC)
    installClusterRole: $(kubeApply .yaml.withClusterRole)
    installMayaAPIServer: $(kubeApply .yaml.withAPIServer)
    installNDM: $(kubeApply .yaml.withNDM)
    installProvisioner: $(kubeApply .yaml.withProvisioner)
    mayaOption: $($opts := kubeKind "deploy" | kubeName "maya-apiserver" | kubePath ".spec.replicas")
    verifyMayaWith3Replicas: $(kubeGetNumericValue $opts | should (haveLen 3))
```

```yaml
kind: MayaScript
spec:
status:
  output:
    title: I would like to manage my kubernetes scripts in a better way
    runs:
    - desc: I have a kubernetes cluster with 3 nodes
      withSCYaml: ...
      withClusterRoleYaml: ...
      withAPIServerYaml: ...
      withNDMYaml: ...
      withProvisionerYaml: ...
    - it: should install openebs without problems
      installSC: ok, 7secs
      installClusterRole: ok, 5secs
      installMayaAPIServer: ok, 2secs
      installNDM: ok, 10secs
      installProvisioner: ok, 5secs
      mayaOption: ok, 1secs
      verifyMayaWith3Replicas: ok, 1secs
```
