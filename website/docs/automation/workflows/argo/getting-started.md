---
title: "Getting Started"
sidebar_position: 5
---

In this scenario you will:

1. Install Argo Workflows into a Kubernetes cluster.
2. Run your first workflow from the user interface.
3. Download, install, and run the Argo CLI.

A workflow is defined as a **Kubernetes resource**. Each workflow consists of one or more templates, one of which is
defined as the entrypoint. Each template can be one of several types, in this example we have one template that is a
container.

```file
manifests/modules/automation/workflows/argo/getting-started/hello-workflow.yaml
```

There are several other types of templates, and we'll come to more of them soon.

Because a workflow is just a Kubernetes resource, you can use `kubectl` with them.

Create a workflow:

```bash
$ kubectl -n argo apply -f ~/environment/eks-workshop/modules/automation/workflows/argo/getting-started/hello-workflow.yaml
```

Then you can wait for it to complete (around 1m):

```bash timeout=120
$ kubectl -n argo wait workflows/hello --for condition=Completed --timeout 2m
```