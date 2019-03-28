### Meta Info
- Version - 4
- Last Updated On - 27-Mar-2019

### Motivation
What things in software are most common? What is the most simplistic version of any software product. IMO it is its CLI.
Somehow I tend to agree that given a CLI that autocompletes & provides me the help at the terminal itself is the coolest
thing and at the same time the most most simplest thing that has ever happened to software. Recently same can be told for
a declarative specification that can be saved, versioned and injected into workflows.

The motivation of this article is to find a balance between action oriented CLI to specification driven yamls.

### What all behaviours do we encounter in our day job (_i.e. Actions / Verbs_)
I shall take the example of openebs storage as the software. What can be various actions related to openebs? We can think
of below:
- create
- delete
- get
- identify
- list
- filter
- resize
- backup
- snapshot
- etc...

### What all nouns do we encounter in our day job (_i.e. Entities_)
Lets think of various nouns used in openebs:
- cstor volume, cstor volume list
- cstor replica, cstor replica list
- cstor target, cstor target list
- cstor pool, cstor pool list
- jiva volume, jiva volume list
- jiva target, jiva target list
- jiva pool, jiva pool list
- etc...

### Mix them up (_typically seen in CLI_)
- create cstor volume
- delete cstor pool
- resize cstor volume
- backup cstor volume
- get cstor volume
- etc...

### When CLI camouflages as a declarative YAML (_i.e. Devil_)
This is considered as the devil by most fellow programmers. However, IMO if it helps users to get the requirement done
in a static compiled language and can be saved & versioned then why not. Being declarative is not a option but a need.
In other words, there would not have any `github actions` to begin with

### Prototype
```yaml
spec:
  create this pod: 
    construct:
      cmd: create kube pod
      with:
        namespace: default
        inCluster: false
        output: ".spec.hostName"
        yaml: blabh blah blah
      if:
        isRunning:
        isLabel: "app=openebs"
    options:
      saveas: createit
      audit: true
      runif: noErrors
  check this pod: 
    construct:
      cmd: get kube pod
      with:
        namespace: default
        name: poddy
        inCluster: false
        output: ".spec.hostName"
      if:
        isRunning:
        isLabel: "app=openebs"
    options:
      saveas: poddy
      audit: true
      runif: noErrors
```

### Prototype - 2
- Yaml when Unmarshaled gets the job done
- Build and Run
- Build options and run options
- Runtask Execute Steps:
  - Go template the cmd
  - Unmarshal // Gets all the job done
  - Repeat next cmd

```yaml
kind: RunTask
spec:
  desc: do my work please
  runs:
  - desc: create this pod
    id: pod101
    cmd:
      type: pod
      action: create
      options: 
      - func: withYaml
        args: |
          kind: Pod
          name: poddie
          namespace: default
      output: // optional
      - alias: uuid
        path: .metadata.uuid
  - desc: get me a list of pods
    id: pod123
    cmd:
      type: podList
      action: list // or apiList, optional
      options: 
      - func: withNamespace 
        args: 
        - default
      - func: inCluster
      checks:
      - func: isRunning
      - func: isNamespace
        args: 
        - default
  - desc: display the output
    id: display101
    cmd:
      type: display
      options: 
      - func: withYaml 
        args: |
          kind: PodList
          items:
          - id: {{ .pod123.id[0] }}
```

### Prototype - 3
- Yaml when unmarshaled gets the job done
- Schema:
  - options, 
  - input, 
  - output
- Can be formed as a list of cmds
- Runtask Execute Steps:
  - Go template the cmd
  - Unmarshal Yaml
  - _NOTE: Unmarshal will do everything_
- Unmarshal Steps:
  - Delegate to respective kind & action
  - Pass the input & output to above delegated object

```yaml
kind: RunTask
spec:
  desc: do my work please
  runs:
  - desc: create this pod
    id: pod101
    options:
      retry: 2, 1s # optional
      isRun: true # optional
      action: Create
      kind: Pod
    input:
    - type: withYaml
      value: |
        kind: Pod
        name: poddie
        namespace: default
    output: # optional
    - type: getPathValueString
      value: .metadata.uuid
      alias: uuid
    - type: getPathValueInt
      value: .metadata.generation
      alias: generation
```