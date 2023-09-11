---
title: "Workflows Examples"
sidebar_position: 20
---

This is a free-form scenario that collects all the [Argo Workflows examples](https://github.com/argoproj/argo-workflows/tree/master/examples) for you and allows you to run them.


- List all examples
```bash
$ ls -l ~/environment/eks-workshop/modules/automation/workflows/argo/examples/
```

- You may need to `kubectl apply` additional files to the cluster prior to running an example:
```bash
$ kubectl apply -n argo -f ~/environment/eks-workshop/modules/automation/workflows/argo/examples/workflow-template/templates.yaml
```

- Run one of the examples
```bash
$ argo submit -n argo --watch ~/environment/eks-workshop/modules/automation/workflows/argo/examples/coinflip.yaml
```
