---
title: "Argo Workflows"
sidebar_position: 1
sidebar_custom_props: {"module": true}
---

:::tip Before you start
Prepare your environment for this section:

```bash timeout=300 wait=30
$ curl -s https://raw.githubusercontent.com/csantanapr/argo-workflows-intro-course/master/argo-workflows/install.sh|sh
```

This will make the following changes to your lab environment:
- Install Argo Events
- Install Argo Workflows

You can view the Terraform that applies these changes [here](https://github.com/VAR::MANIFESTS_OWNER/VAR::MANIFESTS_REPOSITORY/tree/VAR::MANIFESTS_REF/manifests/modules/automation/workflows/argo/.workshop/terraform).

:::

## Argo Workflows
[Argo Workflows](https://github.com/argoproj/argo-workflows) is an open-source, Kubernetes-native workflow engine designed for running complex jobs and orchestrating parallel tasks. It allows users to define, run, and manage workflows directly within their Kubernetes cluster. Argo Workflows provides a rich set of features including DAG-based dependencies, parameterization, and artifact management. It is widely used for CI/CD pipelines, machine learning, and data processing tasks.
![Kubernetes Diagram with Argo Workflows](https://github.com/argoproj/argo-workflows/raw/master/docs/assets/screenshot.png)'





