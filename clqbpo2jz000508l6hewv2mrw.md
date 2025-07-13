---
title: "Compare Kubernetes Manifest files in VSCode"
seoTitle: "KubeMani Diff, VSCode Extension for comparing Kubernetes Manifest"
datePublished: Tue Dec 19 2023 02:13:48 GMT+0000 (Coordinated Universal Time)
cuid: clqbpo2jz000508l6hewv2mrw
slug: compare-kubernetes-manifest-files-in-vscode
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1702951966811/19503b14-0715-4489-a0c7-a82a990d8787.png
tags: kubernetes, vscode-extensions, kubernetes-manifest

---

## Introduction

I recently published a VSCode extension called [`KubeMani Diff`](https://marketplace.visualstudio.com/items?itemName=sunggun.kubemani-diff), which is a tool for comparing Kubernetes manifest files. With this extension, you can easily identify the differences between two Kubernetes manifest files.

`KubeMani Diff` also provides a tree view of the Kubernetes objects present in a long manifest file. This allows you to navigate through the file and view the differences for each object, making it easier to focus on specific differentiation between 2 files.

## Motivation

As a DevOps engineer responsible for managing Kubernetes clusters and their packages as a platform service, it can be challenging to keep track of changes made to Helm Charts, Kustomizations, and Kubernetes Manifests. This becomes especially difficult when dealing with significant version differences or when changes are made to the source of the Chart or Manifest.

I struggled to track differences when attempting to use the built-in diff tool in VSCode, combined with other extensions for sorting YAML in long manifest files exceeding 20,000 lines.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702939857737/496677a0-820b-4b2d-b8aa-e4c7156abb3e.gif align="center")

KubeMani Diff addresses this challenge by providing a visual diff view that highlights discrepancies between two manifest files, making it easier to understand and analyze the differences.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702940474232/7014ad62-b00f-4e87-84ad-13342913fe65.gif align="center")

## Use Cases

Here is a real-world use case that I was eager to solve. So, my team deployed ArgoCD in the cluster using a Helm Chart made from the manifest downloaded from the source repository. We customized it further using Kustomize and added extra resources as part of the Helm template. Whenever we needed to upgrade the customized Helm Chart of ArgoCD, we repeated this process. However, as the version grew, we had to make additional changes in Kustomization. Moreover, there were many steps to follow for upgrading, and we were not confident if anyone made direct changes to the downloaded manifest. Later on, we discovered that the official Helm Chart covers most of the customizations we made with Kustomization. Consequently, we were able to create our Helm Chart with a sub-chart with the original chart. However, we still lacked confidence in moving forward with the new way. To build team confidence, we need to compare the Kubernetes Manifest results from the old and new ways and make adjustments to make them as similar as possible.

## Usage

To use KubeMani Diff, you need to prepare and save the Kubernetes manifest files you want to compare. You can use the command below to generate the files from Helm Charts and Kustomization:

```bash
helm template argocd \
  argo/argo-cd \
  --namespace argocd \
  --version=5.51.6
> argocd-helm.yaml
```

```bash
kustomize build > argocd-kustomization.yaml
```

After saving the files, open them in VSCode and select two files. Right-click on the selected files, and choose "Select Kubernetes manifest files to compare".

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702950037342/e7fd7078-1c83-4bf4-a79d-efb3dcf4bd0b.png align="center")

You can use the tree view to compare the structure of an object. If the object is only on one side, the tree view will indicate it with an A or B icon. If the object is on both sides, it will show an AB icon. Once you click on the object, the tree view will provide a diff view.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702949274580/8825fe58-639e-4579-855c-0c3a4acd4fec.png align="center")

## Installation

You can search by keyword for Kubernetes manifest diffs, or find "kubemani-diff" in the VSCode extension menu. Alternatively, visit the [Visual Studio Code Marketplace](https://marketplace.visualstudio.com/items?itemName=sunggun.kubemani-diff) to install the extension and leave comments and ratings for improvements.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1702950956409/45999b6a-df6d-4925-9267-062e4fbaf9c3.png align="center")

Thanks for reading!