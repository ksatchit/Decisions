### OpenEBS Maya - Version 5
Exposing CASTemplate along with RunTasks enables exposing **logic in a declarative manner**. It helped the team in numerous
ways. One of the typical use of these templates was the ability to embed logic into RunTasks and solve the problem statement
externally without writing code.

### Is Templating & Template Functions Good?
```yaml
- Template functions can be made similar to invoking CLI commands
  - The template will look like a scripted copy of bunch of CLI commands on terminals
- Template functions can be much more powerful when
  - We can join multiple operations
  - We can operate these functions selectively
  - We can keep piping the results to new functions with ease
- Demerit
  - The execution control is entirely with template runner
  - Erroring out of this template runner is not elegant
  - Debugging is pathetic
```

### The way forward
```yaml
- Avoid the need to write specific engines that wrap template runner
- Ability to Unit Test a RunTask
- Reduce the need for programming in templates
  - In other words convert the template into a spec
- Avoid verbosity in RunTasks
- Make the template intuitive & simple to learn
- Improve the debuggability aspect
- Avoid the need for multiple RunTasks
- Ability to execute RunTasks conditionally
- Ability to add skip, validate, error injection & non-functional logic
```

### Next Steps
Maya design will expose two new kinds of config in addition to template based config. 
They are explained below:
- human config - human understood and are inline with the feature
- raw config - program understood that implements a feature

A raw config looks like a pseudo-code. A human based config is typically implemented (or transformed & then run) as a raw
config.

#### Package Design:
```
  - spec - pkg/apis/openebs.io/v1alpha1/
  - engine - pkg/engine/v1alpha1/
  - runner - pkg/task/v1alpha1/ vs. pkg/runner/v1alpha1
  - resource - pkg/resource/v1alpha1/
  - client - pkg/client/k8s/v1alpha1/
  - http client - pkg/client/http/v1alpha1/
  - template - pkg/template/v1apha1/
  - install - pkg/install/v1alpha1/
  - volume - pkg/volume/v1alpha1/
  - snapshot - pkg/snapshot/v1alpha1/
  - clone - pkg/clone/v1alpha1/
  - pool - pkg/pool/v1alpha1/
```

#### Low Level Design:
- engine
```go
type Interface interface {}
type casTemplateEngine struct {
  data        interface{}
  provider    CASTemplateProvider
  rtEngine    RunTaskEngine
}
type runTaskEngine struct {
  config    map[string]interface{}
  data      interface{}
  provider  RunTaskProvider
  runner    CommandRunner
}
```
- runner
```go
type CommandRunner interface {}
type Command struct {}
func NewCommand() *Command {}
func WithOptions(o CommandOptions) cmd.Interface {}
type CommandOptions struct {
  Select    Selector
  Store     Storage
  WillRun   RunCondition
}
type RunCommands []RunCommand
type StoreCommand struct {}
type templateRunner struct {}
type runner struct {}
type repeater struct {}
```
- resource
```go
type Interface interface {}
type Provider interface {}
type Getter interface {}
type Updater interface{}
type Applier interface{}

type K8sResource struct {}
type CASTemplate struct {}
type RunTask struct {}
type ConfigMap struct{}

type Namer interface {
  Name() (string, error)
}
type name struct {}
type nameBySC struct {}
type nameByENV struct {}
```

### Template Config - Futuristic Version i.e. similar to **MySQL Query Language**
This rough work lists down all sorts of templating possibilities for a `template config`. 

_NOTE: This will get refined further based on feedbacks, experiences & my brain's biasedness._
_NOTE: The core structures/code will be reused to implement **Human Config** & **Raw Config**._

#### Select Clause
- [ ] `select all | create kubernetes service | spec $yaml | totemplate .Values .Config | run`
- [ ] `select all | text template .Values $doc | tounstruct | run | saveas "123" .Values`

#### Where Clause
- [ ] `select "name" "ip" | get k8s service | where "name" $name | run`
- [ ] `select "name" "ip" | get k8s service | where "name" "ne" $name | run`
- [ ] `select "name" "ip" | delete k8s service | where "name" $name | run`
- [ ] `select "name" "ip" | list k8s service | where "name" "ne" $name | run`

#### Whereop Clause
- [ ] `select "name" "ip" | list k8s service | where "name" $name | where "label" $app | whereop "any" | run`
- [ ] `select "name" "ip" | list k8s service | where "name" $name | where "label" $app | whereop "all" | run`
- [ ] `select "name" "ip" | list k8s service | where "name" $name | where "label" $app | whereop "atleasttwo" | run`

#### When Clause
- [ ] `select "name" "ip" | get k8s service | where "name" "ne" $name | when eq $willrun "true" | run`

#### Join multiple queries
- [ ] `select name, ip | get k8s svc | where label eq abc | join list k8s pod | where "label" "all"`

#### select, from, groupby
- [ ] `select "name" "namespace" | from .TaskResult.101 | runas 201`
- [ ] `select "name" "namespace" | from .TaskResult.101 .TaskResult.102 | groupby "name" | runas 202`
- verify if https://github.com/Masterminds/sprig/blob/master/dict.go will solve the purpose !!!

```go
type row map[string]interface{}
type table []row
```

#### Error Handling Clause
- Should assert & expect like things be exposed ?
  - https://github.com/kubernetes-csi/csi-test/blob/master/pkg/sanity/controller.go


#### Set Clause
- [ ] `create k8s service | spec $yaml | totemplate .Values | set "namespace" $ns | run`
- [ ] `create jiva snapshot | set "spec" $yaml | txttemplate .Values | run`
- [ ] `create jiva snapshot | set "name" $name | set "capacity" "2G" | run`
- [ ] `create k8s pod | set spec $spec | run`
- [ ] `creates k8s pod | set image $image | run`
- [ ] `create k8s jiva replica | set image $image | run`
- [ ] `delete jiva data | run`
- [ ] `create jiva target | set image $image | run`
- [ ] `create jiva target | set "container.path" $container | run`
- [ ] `create jiva target | set "spec" $spec | run`
- [ ] `create jiva target | set "doc" $doc | run`
- [ ] `create jiva target | set "yaml" $yaml | run`
- [ ] `create k8s deploy | set "yaml" $yaml | txttemplate .Values .Config | run`
- [ ] `create k8s deploy | set "yaml" $yaml | set "image" $image | txttemplate .Values .Config | run`

#### Set Conditionally
- [ ] `create k8s service | spec $yaml | totemplate .Values | setif isnamespace "namespace" "value" | run`


#### Text Template as a template function
- [ ] `$doc | exec template . Values | run`
- [ ] `$doc | exec text template | data . Values | run`
- [x] `$doc | text template | data . Values | run`
- [ ] `create kubernetes service | specs $doc | txttemplate . Values | run`
- [ ] `create k8s svc | spec $doc | txttemplate .Volume .Config  | run`
- [x] `select name, ip | create k8s service | spec $doc | totemplate .Volume .Config | run`

#### Accept docs
```
# unmarshall from a json or a yaml doc in string format to []byte to interface{}
unmarshall .Value.docs.abc01 | runas "abc01" $runner
unmarshall .Value.docs.abc02 | runas "abc02" $runner

# first run the template with all available values & return as a string
$doc := txtTemplate .Value.docs.abc01 . | runas "tpl" $runner
# now unmarshall it to []byte to interface{}
unmarshall .TaskResult.tpl.result | runas "deploy" $runner

create k8s deploy | withoption "body" .TaskResult.deploy.result | runas "createK8sDeploy" $runner
```

#### Sample Template Config
```yaml
apiVersion: openebs.io/v1alpha1
kind: RunTask
metadata:
  name: jiva-volume-delete-deletedata-default-0.7.0
spec:
  meta: |
    id: deljivadata
    kind: Command
  post: |
    {{- $base := print "http://" .TaskResult.deletelistsvc.ips ":9501" -}}
    {{- delete jiva volume | url $base | name .Volume.owner | run | saveas "deljivadata" .TaskResult -}}
    {{- $err := toString .TaskResult.deljivadata.error -}}
    {{- $err | empty | not | verifyErr $err | saveIf "deljivadata.verifyErr" .TaskResult | noop -}}
```

### Raw Config Samples
- **flavour 1 - pure yaml -- with functions sprinkled around**
  - entire options is optional
  - options is populated by the controller
  - run id can be auto generated
```yaml
kind: RunTask
options:
  values:
  - svc: my-service
  - deploy: my-deploy
spec:
  runs:
  - let: name, ns
    value: .metadata.name, .metadata.namespace
  - let: svc, deploy
    value: {{values.svc}} | default "mysvc", {{values.deploy}}
  - let: funky
    func: $mysvc | suffix "-svc" | prefix "my-"
  - select: $name $ns
    get: kube-service
    where: labels.name.eq.John
    with: 
    - name: $svc
  - select: $name $ns
    get: kube-deploy
    with:
    - name: {{values.deploy}}
status:
  select: all
  get: runtask-result
```
- **flavour 2 - functions && pipes**
```yaml
Kind: RunTask
spec:
  options:
    runner: storeRunner
    runIf:  ifNoErr
    values: <map[string]interface{}>
  runs:
  - id: prep
    let: .Values.name | default "none" | alias "var" "named"
  - id: mySvc
    run: select "clusterIP" | create kube service | with "name" .var.named
  - id: myDeploy
    run: select ".namespace.name" | create kube deploy | with "name" "my-deploy"
  status:
    run: select "all" | get runtask result
```
- **flavour 3 - functions & pipes**
  - entire options is optional
  - options is populated by the controller
  - run id can be auto generated
```yaml
Kind: RunTask
spec:
  runs:
  - let: .Values.name | default "none" | alias "names" "named"
  - let: .Values.deploy.name | default "none" | alias "mydep"
  - run: select "clusterIP" | create kube service | with "name" .names.named
  - run: select ".namespace.name" | create kube deploy | with "name" .var.mydep
  status:
    run: select "all" | get runtask result
```
### Human Config Samples
- a sample blueprint
```yaml
work:
  flow:
  - abc
  - def
  abc:
  def:
```
- a install spec
```yaml
kind: Install
spec:
  # ordered options that influences the installation workflow
  options:
  - labels
  - namespace
  - artifacts
  # labels related options/configs
  labels:
    add: app=cool
    update: openebs.io/version=1.7
  # artifacts related options/configs
  artifacts:
    include: my_artifact
    exclude: his_artifact
  # namespace related options/configs
  namespace:
    useOpenEBS: true
```

### Wild Ideas
- Replace go text templating via:
  - https://github.com/tidwall/sjson
- Patch/Update json via:
  - https://github.com/tidwall/sjson
- input & output of parsing a json via json path is json
  - https://github.com/jmespath/go-jmespath
- extracting values from json as json
  - https://github.com/jmespath/go-jmespath
- merge json:
  - https://github.com/evanphx/json-patch/blob/master/merge_test.go
- patch json i.e. operation based merge ?
  - https://github.com/evanphx/json-patch/blob/master/patch_test.go
- Use this to replace go text templating
  - https://github.com/evanphx/json-patch/blob/master/patch_test.go
  - The patch operation & path specific details will be hidden/internal to your custom library
  - Just the doc hidden in RunTask or defaults to common stuff in .go file
  - The merge == pure json as config of CASTemplate
- Making `select` more powerful:
  - https://github.com/jmespath/go-jmespath/tree/master/compliance

### Useful References:
- Naming References - https://concourse-ci.org/reference.html
- Diagram References - https://medium.com/bluecore-engineering/were-all-using-airflow-wrong-and-how-to-fix-it-a56f14cb0753
