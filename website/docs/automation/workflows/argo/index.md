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
- Install Argo Workflows


:::

## Argo Workflows
[Argo Workflows](https://github.com/argoproj/argo-workflows) is an open-source, Kubernetes-native workflow engine designed for running complex jobs and orchestrating parallel tasks. It allows users to define, run, and manage workflows directly within their Kubernetes cluster. Argo Workflows provides a rich set of features including DAG-based dependencies, parameterization, and artifact management. It is widely used for CI/CD pipelines, machine learning, and data processing tasks.

![Argo Workflows UI](https://github.com/argoproj/argo-workflows/raw/master/docs/assets/screenshot.png)'

![Argo Workflows Architecture](https://argoproj.github.io/argo-workflows/assets/diagram.png)





