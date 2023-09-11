---
title: "Templates"
sidebar_position: 10
---


In this scenario you will:

1. Learn about the different types and different **categories of template**.
2. Learn how to template your templates using **template tags**.
3. Combine them together to make **complex workflows**.

From now on, Argo Workflows will be set up automatically for you. Wait until you see "Ready" in the console.

There will also be optional exercises. These will mean the lessons take longer, but help you be sure you've really understood everything.

## Templates Types
There are several types of templates, divided into two different categories: **work** and **orchestration**.

The first category defines **work** to be done. This includes:

* Container
* Container Set
* Data
* Resource
* Script

The second category **orchestrates** the work:

* DAG
* Steps
* Suspend


## Container Templates
A container template is the most common type of template, lets look at a complete example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: container-
spec:
  entrypoint: main
  templates:
  - name: main
    container:
      image: docker/whalesay
      command: [cowsay]
      args: ["hello world"]
```

Let's run the workflow:

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/templates/container-workflow.yaml
```

Open the Argo Workflows UI. Then navigate to the workflow, you should see a single container running.

## Exercise

Edit the workflow to make it echo "howdy world".

## Template Tags

**Template tags** (also known as **template variables**) are a way for you to substitute data into your workflow at runtime. Template tags are delimited by `{{`
and `}}` and will be replaced when the template is executed.

What tags are available to use depends on the template type, and there are a number of global ones you can use, such as `{{workflow.name}}`, which is replaced by the workflow's name:

```yaml
    - name: main
      container:
        image: docker/whalesay
        command: [ cowsay ]
        args: [ "hello {{workflow.name}}" ]
```

Look at the full example:
```bash
$ cat ~/environment/eks-workshop/modules/automation/workflows/argo/templates/template-tag-workflow.yaml
```

Submit this workflow:
```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/templates/template-tag-workflow.yaml
```


You can see the output by running
```bash
$ argo logs @latest
```


You should see something like:

```shell
 __________________________
< hello template-tag-kqpc6 >
 --------------------------
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

There are many more different tags, you can [read more about template tags in the docs](https://argoproj.github.io/argo-workflows/variables/).

## Exercise

Change the workflow to echo the date the workflow was created.


## Work Templates
What other types of *work* templates are there?

A **container set** allows you to run multiple containers in a single pod. This is useful when you want the containers
to share a common workspace, or when you want to consolidate pod spin-up time into one step in your workflow.

A **data template** allows you get data from storage (e.g. S3). This is useful when each item of data represents an item
of work that needs doing.

A **resource template** allows you to create a Kubernetes resource and wait for it to meet a condition (e.g. successful)
. This is useful if you want to interoperate with another Kubernetes system, like AWS Spark EMR.

A **script template** allows you to run a script in a container. This is very similar to a container template, but when
you've added a script to it.

Every type of template that does work does it by running a pod. You can use `kubectl` to view these pods:

```bash
$ kubectl get pods -l workflows.argoproj.io/workflow
```


You can identify workflow pods by the `workflows.argoproj.io/workflow` label.

You should see something like this:

```shell
NAME                 READY   STATUS      RESTARTS   AGE
container-m5664      0/2     Completed   0          5m21s
template-tag-kqpc6   0/2     Completed   0          4m6s
```

## Exercise

Use `kubectl describe` to describe a workflow pod. What interesting information is contained within the pod labels
and annotations?

## DAG Template
A **DAG template** is a common type of *orchestration* template.
Let's look at a complete example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: dag-
spec:
  entrypoint: main
  templates:
    - name: main
      dag:
        tasks:
          - name: a
            template: whalesay
          - name: b
            template: whalesay
            dependencies:
              - a
    - name: whalesay
      container:
        image: docker/whalesay
        command: [ cowsay ]
        args: [ "hello world" ]

```

In this example, we have two templates:

* The "main" template is our new DAG.
* The "whalesay" template is the same template as in the container example.

The DAG has two tasks: "a" and "b". Both run the "whalesay" template, but as "b" depends on "a", it won't start until "
a" has completed successfully.

Let's run the workflow:

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/templates/dag-workflow.yaml
```


You should see something like:

```shell
STEP          TEMPLATE  PODNAME              DURATION  MESSAGE
 ✔ dag-shxn5  main
 ├─✔ a        whalesay       dag-shxn5-289972251  6s
 └─✔ b        whalesay       dag-shxn5-306749870  6s
```

Did you see how `b` did not start until `a` had completed?

Open the Argo Server tab and navigate to the workflow, you should see two containers.

## Exercise

Add a new task named "c" to the DAG. Make it depend on both "a" and "b".
Go to the UI and view your updated workflow graph.


## Loops

The ability to run large parallel processing jobs is one of the key features of Argo Workflows.
Let's have a look at using loops to do this.

## withItems

A DAG allows you to loop over a number of items using `withItems`:

```yaml
      dag:
        tasks:
          - name: print-message
            template: whalesay
            arguments:
              parameters:
                - name: message
                  value: "{{item}}"
            withItems:
              - "hello world"
              - "goodbye world"
```

In this example, it will execute once for each of the listed items. We can see a **template tag** here. `{{item}}` will be replaced with "hello world" and "goodbye world". DAGs execute in parallel, so both tasks will be started at the same time.

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/templates/with-items-workflow.yaml
```

You should see something like:

```shell
STEP                                 TEMPLATE  PODNAME                      DURATION  MESSAGE
 ✔ with-items-4qzg9                  main
 ├─✔ print-message(0:hello world)    whalesay  with-items-4qzg9-465751898   7s
 └─✔ print-message(1:goodbye world)  whalesay  with-items-4qzg9-2410280706  5s
```

Notice how the two items ran at the same time.

## withSequence

You can also loop over a sequence of numbers using `withSequence`:

```yaml
      dag:
        tasks:
          - name: print-message
            template: whalesay
            arguments:
              parameters:
                - name: message
                  value: "{{item}}"
            withSequence:
              count: 5
```

As usual, run it:
```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/templates/with-sequence-workflow.yaml
```


```shell
STEP                     TEMPLATE  PODNAME                         DURATION  MESSAGE
 ✔ with-sequence-8nrp5   main
 ├─✔ print-message(0:0)  whalesay  with-sequence-8nrp5-3678575801  9s
 ├─✔ print-message(1:1)  whalesay  with-sequence-8nrp5-1828425621  7s
 ├─✔ print-message(2:2)  whalesay  with-sequence-8nrp5-1644772305  13s
 ├─✔ print-message(3:3)  whalesay  with-sequence-8nrp5-3766794981  15s
 └─✔ print-message(4:4)  whalesay  with-sequence-8nrp5-361941985   11s
```

See how `5` pods were run at the same time, and that their names have the item value in them, zero-indexed?

## Exercise

Change the **withSequence** to print the numbers 10 to 20.


## Workflow Templates

**Workflow templates** (not to be confused with a template) allow you to create a library of code that can be reused.
They're similar to pipelines in Jenkins.

Workflow templates have a different `kind` to a workflow, but are otherwise very similar:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: hello
spec:
  entrypoint: main
  templates:
    - name: main
      container:
        image: docker/whalesay
        command: [ cowsay ]
```

Let's create this workflow template:

```bash
$ argo template create ~/environment/eks-workshop/modules/automation/workflows/argo/templates/hello-workflowtemplate.yaml
```


You can also manage templates using `kubectl`:

```bash
$ kubectl apply -f ~/environment/eks-workshop/modules/automation/workflows/argo/templates/hello-workflowtemplate.yaml
```

This allows you to use GitOps to manage your templates.

To submit a template, you can use the UI or the CLI:

```bash
$ argo submit --watch --from workflowtemplate/hello
```


You should see:

```shell
STEP            TEMPLATE  PODNAME      DURATION  MESSAGE
 ✔ hello-c622t  main      hello-c622t  33s
```

Lets take a look at the workflow you created:

```bash
$ argo get @latest -o yaml
```

Look for the workflow specification in the output:

```yaml
spec:
  workflowTemplateRef:
    name: hello
```

Note how the specification of the workflow is actually a reference to the template.

## Exercise

* Use the user interface to submit a workflow template:
* Update the workflow template to add some parameters (e.g. to print a message). Use `argo submit --from` to submit it
  with different parameters.


## Cron Workflows
A **cron workflow** is a workflow that runs on a cron schedule:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: CronWorkflow
metadata:
  name: hello-cron
spec:
  schedule: "* * * * *"
  workflowSpec:
    entrypoint: main
    templates:
      - name: main
        container:
          image: docker/whalesay
```

When it should be run is set in the `schedule` field, in the example every minute.

Lets created this cron workflow:

```bash
$ argo cron create ~/environment/eks-workshop/modules/automation/workflows/argo/templates/hello-cronworkflow.yaml
```

You'll need to wait for up to a minute to see the workflow run.

## Exercise

Cron workflows can be submitted immediately from the CLI or the UI. Find out how.




## Orchestration Templates
We learned that a DAG template is a type of *orchestration* template. What other types of *orchestration* templates are there?

A **steps template** allows you to run a series of steps in sequence.

A **suspend template** allows you to automatically suspend a workflow, e.g. while waiting on manual approval, or while
an external system does some work.

Orchestration templates **do NOT** run pods.
You can check by running `kubectl get pods`

## Exit Handler
If you need to perform a task after something has finished, you can use an exit handler. Exit handlers are specified
using `onExit`:

```yaml
      dag:
        tasks:
          - name: a
            template: whalesay
            onExit: tidy-up
```

They just state the name of the template that should be run on-exit.

Lets look at a complete example:

```bash
$ cat ~/environment/eks-workshop/modules/automation/workflows/argo/templates/exit-handler-workflow.yaml
```


Run it:

```bash
$ argo submit --watch ~/environment/eks-workshop/modules/automation/workflows/argo/templates/exit-handler-workflow.yaml
```

You should see:

```shell
STEP                   TEMPLATE  PODNAME                        DURATION  MESSAGE
 ✔ exit-handler-plvg7  main
 ├─✔ a                 whalesay  exit-handler-plvg7-1651124468  5s
 └─✔ a.onExit          tidy-up   exit-handler-plvg7-3635807335  6s
```

Note how the exit handler task ran last.

## Exercise

An exit handler can be run at the end of a template, or at the end of a workflow. Change the example to run one at the
end of the workflow.

Learn more about [exit handlers](https://argoproj.github.io/argo-workflows/walk-through/exit-handlers/), as well as their close cousin, [lifecycle hooks](https://argoproj.github.io/argo-workflows/lifecyclehook/), in the Argo Workflows documentation.


# Conclusion
Let's recap:

* A workflow consists of one or more templates.
* One template is the **entrypoint** and it runs first.
* Some templates do **work**, such as a container templates.
* Other templates **orchestrate** that work, such as DAG templates.
* You can run tasks at the end of a template or workflow using an **exit handler**.

Please [let us know what can be improved](https://github.com/csantanapr/argo-workflows-intro-course/issues).
