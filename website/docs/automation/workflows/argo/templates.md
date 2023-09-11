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

