---
title: "Inputs and Outputs"
sidebar_position: 15
---

In this scenario you will:

1. Learn about **parameters**.
1. Create a workflow that uses parameters
1. Learn about **artifacts** and storage.
1. Create a workflow that generates and consumes artifacts.

## Parameters

One type of input or output is a **parameter**. Unlike artifacts, these are plain string values, and are useful for most
simple cases.

## Input Parameters

Let's have a look at an example:

```yaml
    - name: main
      inputs:
        parameters:
          - name: message
      container:
        image: docker/whalesay
        command: [ cowsay ]
        args: [ "{{inputs.parameters.message}}" ]
```

This template declares that it has one input parameter named "message".

See the complete workflow:

```bash
$ cat  ~/environment/eks-workshop/modules/automation/workflows/argo/inputs-and-outputs/input-parameters-workflow.yaml
```


See how the workflow itself has arguments?

Run it:

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/inputs-and-outputs/input-parameters-workflow.yaml
```

You should see:

```shell
STEP                       TEMPLATE  PODNAME                 DURATION  MESSAGE
 ✔ input-parameters-mvtcw  main      input-parameters-mvtcw  8s
```

If a workflow has parameters, you can change the parameters using `-p` using the CLI:

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/inputs-and-outputs/input-parameters-workflow.yaml -p message='Welcome to Argo!'
```


You should see:

```shell
STEP                       TEMPLATE  PODNAME                 DURATION  MESSAGE
 ✔ input-parameters-lwkdx  main      input-parameters-lwkdx  5s
```

Let's check the output in the logs:

```bash
$ argo logs @latest
```

You should see:

```shell
 ______________
< Welcome to Argo! >
 --------------
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
```

## Output Parameters

Output parameters can be from a few places, but typically the most versatile is from a file. In this example, the
container creates a file with a message in it:

```yaml
  - name: whalesay
    container:
      image: docker/whalesay
      command: [sh, -c]
      args: ["echo -n hello world > /tmp/hello_world.txt"]
    outputs:
      parameters:
      - name: hello-param
        valueFrom:
          path: /tmp/hello_world.txt
```

In a DAG template and steps template, you can reference the output from one task, as the input to another
task using a **template tag**:

```yaml
      dag:
        tasks:
          - name: generate-parameter
            template: whalesay
          - name: consume-parameter
            template: print-message
            dependencies:
              - generate-parameter
            arguments:
              parameters:
                - name: message
                  value: "{{tasks.generate-parameter.outputs.parameters.hello-param}}"
```

See the complete workflow:

```bash
$ cat ~/environment/eks-workshop/modules/automation/workflows/argo/inputs-and-outputs/parameters-workflow.yaml
```

Run it:

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/inputs-and-outputs/parameters-workflow.yaml
```

You should see:

```shell
STEP                     TEMPLATE       PODNAME                      DURATION  MESSAGE
 ✔ parameters-vjvwg      main
 ├─✔ generate-parameter  whalesay       parameters-vjvwg-4019940555  43s
 └─✔ consume-parameter   print-message  parameters-vjvwg-1497618270  8s
```

Learn more about parameters in the Argo Workflows documentation:
- [Parameters overview](https://argoproj.github.io/argo-workflows/walk-through/parameters/)
- [Workflow input parameters](https://argoproj.github.io/argo-workflows/workflow-inputs/).


## Artifacts

Artifact is a fancy name for a file that is compressed and stored in object storage (S3, MinIO, etc.).

There are two kind of artifacts in Argo:

* An **input artifact** is a file downloaded from storage (i.e. S3) and mounted as a volume within the container.
* An **output artifact** is a file created in the container that is uploaded to storage.

Artifacts are typically uploaded into a bucket within some kind of storage such as AWS S3. We call that storage an
**artifact repository**. Within these lessons we'll be using MinIO for this purpose, but you can just imagine it is S3
or what ever you're used to.

## Output Artifacts

Each task within a workflow can produce output artifacts. To specify an output artifact, you must include `outputs` in
the manifest. Each output artifact declares:

* The **path** within the container where it can be found.
* A **name** so that it can be referred to.

```yaml
    - name: save-message
      container:
        image: docker/whalesay
        command: [ sh, -c ]
        args: [ "cowsay hello world > /tmp/hello_world.txt" ]
      outputs:
        artifacts:
          - name: hello-art
            path: /tmp/hello_world.txt
```

When the container completes, the file is copied out of it, compressed, and stored.

Files can also be directories, so when a directory is found all the files are compressed into an archive and stored.

## Input Artifacts

To declare an input artifact, you must include `inputs` in the manifest. Each input artifact must declare:

* Its **name**
* The **path** where it should be created

```yaml
    - name: print-message
      inputs:
        artifacts:
          - name: message
            path: /tmp/message
      container:
        image: docker/whalesay
        command: [ sh, -c ]
        args: [ "cat /tmp/message" ]
```

If the artifact was a compressed directory, it will be uncompressed and unpacked into the path.

## Inputs and Outputs

You can't use inputs and outputs in isolation, you need to combine them together using either a steps or a DAG template, as in the below example:

```yaml
    - name: main
      dag:
        tasks:
          - name: generate-artifact
            template: save-message
          - name: consume-artifact
            template: print-message
            dependencies:
              - generate-artifact
            arguments:
              artifacts:
                - name: message
                  from: "{{tasks.generate-artifact.outputs.artifacts.hello-art}}"
```

In the above example, `arguments` is used to declare the value for the artifact input. This uses a **template tag**. In
this example, `{{tasks.generate-artifact.outputs.artifacts.hello-art}}` becomes the path of the artifact in the
repository.

The task `consume-artifact` must run after `generate-artifact`, so we use `dependencies` to declare that relationship.

Let's see the complete DAG workflow:

```bash
$ cat ~/environment/eks-workshop/modules/automation/workflows/argo/inputs-and-outputs/artifacts-workflow.yaml
```

Let's run an example:

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/inputs-and-outputs/artifacts-workflow.yaml
```

You should see:

```shell
STEP                    TEMPLATE       PODNAME                     DURATION  MESSAGE
 ✔ artifacts-qvcpn      main
 ├─✔ generate-artifact  save-message   artifacts-qvcpn-3260493969  7s
 └─✔ consume-artifact   print-message  artifacts-qvcpn-2991781604  8s
```
