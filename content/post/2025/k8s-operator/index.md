---
title: Operator 开发入门系列（一）：Hello World!
description: 了解 Kubernetes Operator 的基本概念，并学习如何使用 Kubebuilder 创建一个简单的 `Hello World` Operator。 本教程详细介绍了环境准备、项目初始化、CRD 定义、Reconcile 逻辑实现以及 Operator 部署和验证的步骤，帮助你快速入门 Operator 开发。
slug: operator
date: 2025-04-01 08:00:00+0000
image: cover.png
categories: [ devops ]
tags: [ Kubernetes, Operator ]
weight: 1
---

## 背景

我们公司最近计划将产品迁移到 Kubernetes 环境。 为了更好地管理和自动化我们的应用程序，我们决定使用 Kubernetes Operator。 本系列博客将记录我们学习和开发 Operator 的过程，希望能帮助更多的人入门 Operator 开发。

## 目标读者

*   对 Kubernetes 有一定了解的开发人员和运维人员
*   希望使用 Operator 自动化管理应用程序的人员
*   对 Go 语言有基本了解的人员

## 准备工作

在开始之前，你需要准备以下环境：

*   **Go 语言环境 (>= 1.23):** Operator 通常使用 Go 语言开发，你需要安装 Go 语言环境。 建议使用 Go 1.21 或更高版本。 可以从 [https://go.dev/dl/](https://go.dev/dl/) 下载安装包。 安装完成后，请配置好 `GOPATH` 和 `PATH` 环境变量。

*   **Kubernetes 集群:** 你需要一个可用的 Kubernetes 集群来部署和测试 Operator。 可以使用 Minikube、Kind 或其他的 Kubernetes 发行版。

*   **kubectl 命令行工具:** `kubectl` 是 Kubernetes 的命令行工具，用于与 Kubernetes 集群交互。 请确保你已经安装并配置了 `kubectl`， 并且能够连接到你的 Kubernetes 集群。

*   **Kubebuilder (>= 3.0):** Kubebuilder 是一个用于快速构建 Kubernetes Operator 的框架。 使用 Kubebuilder 可以简化Operator 的开发流程，并生成一些必要的代码框架。 可以使用以下命令安装 Kubebuilder：

```bash
cd $HOME/go/bin
curl -L -o kubebuilder "https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)"
chmod +x kubebuilder
```

> 请确保 `$HOME/go/bin` 目录在你的 `PATH` 环境变量中。 可以运行 `kubebuilder version` 命令来验证 Kubebuilder 是否安装成功。

*   **Docker (可选):** 如果你需要构建 Operator 的 Docker 镜像，你需要安装 Docker。

> 我的环境是 MacOS(arm64) + Orbstack

## 什么是 Operator？

简单来说，Operator 是 Kubernetes 的扩展，它利用自定义资源（Custom Resources, CRs）来自动化管理应用程序。Operator 允许我们像管理 Kubernetes 内置资源一样管理复杂的应用程序，例如数据库、消息队列等。

## 为什么选择 Operator？

Operator 提供了一种声明式的方式来管理应用程序的生命周期，包括部署、升级、备份、恢复等。它可以简化运维流程，提高自动化程度，并确保应用程序的状态符合预期。

## 我们的第一个 Operator：Hello World

这个 Operator 将监听一个名为 `HelloWorld` 的自定义资源，并在 Kubernetes 中创建一个 Pod，该 Pod 运行一个简单的 "Hello World" 应用程序。

### 1. 初始化 Kubebuilder 项目

首先，我们需要使用 Kubebuilder 创建一个新的项目。 在你的 `GOPATH` 目录下创建一个新的目录，例如 `hello-world-operator`，然后进入该目录，运行以下命令

```bash
kubebuilder init --domain infini.cloud --repo github.com/infinilabs/hello-world-operator
```

这个命令会创建一个新的 Kubebuilder 项目，并生成一些必要的文件和目录。

### 2. 创建自定义资源（Custom Resource Definition, CRD）

接下来，我们需要定义 `HelloWorld` 资源的结构。 运行以下命令

```bash
kubebuilder create api --group example --version v1alpha1 --kind HelloWorld
```

这个命令会创建一个新的 API 定义，包括 `api/v1alpha1/helloworld_types.go` 和 `controllers/helloworld_controller.go` 两个文件。

编辑 `api/v1alpha1/helloworld_types.go` 文件，修改 `HelloWorldSpec` 的定义，添加 `name` 和 `message` 字段：

```go
// HelloWorldSpec defines the desired state of HelloWorld
type HelloWorldSpec struct {
    // INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
    // Important: Run "make" to regenerate code after modifying this file

    // Name is the name of the HelloWorld resource
    Name string `json:"name,omitempty"`

    // Message is the message to be printed by the pod
    Message string `json:"message,omitempty"`
}
```

### 3. 实现 Reconcile 逻辑

编辑 `controllers/helloworld_controller.go` 文件，实现 `Reconcile` 函数， 创建一个 Pod，该 Pod 运行一个 `busybox` 镜像，并输出 `HelloWorld` 资源中定义的 `message`。

```go
package controllers

import (
    "context"
    "fmt"

    corev1 "k8s.io/api/core/v1"
    apierrors "k8s.io/apimachinery/pkg/api/errors"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/log"

    examplev1alpha1 "github.com/infinilabs/hello-world-operator/api/v1alpha1"
)

// HelloWorldReconciler reconciles a HelloWorld object
type HelloWorldReconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

//+kubebuilder:rbac:groups=example.com,resources=helloworlds,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=example.com,resources=helloworlds/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=example.com,resources=helloworlds/finalizers,verbs=update
//+kubebuilder:rbac:groups=core,resources=pods,verbs=get;list;watch;create;update;patch;delete

// Reconcile is part of the main kubernetes reconciliation loop which aims to
// move the current state of the cluster closer to the desired state.
// For more details, check Reconcile and its Result here:
// - https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.13.0/pkg/reconcile
func (r *HelloWorldReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := log.FromContext(ctx)

    // 1. Fetch the HelloWorld instance
    helloWorld := &examplev1alpha1.HelloWorld{}
    err := r.Get(ctx, req.NamespacedName, helloWorld)
    if err != nil {
        if apierrors.IsNotFound(err) {
            // Object not found, return.  Created objects are automatically garbage collected.
            // For additional cleanup logic use finalizers.
            log.Info("HelloWorld resource not found. Ignoring since object must be deleted")
            return ctrl.Result{}, nil
        }
        // Error reading the object - requeue the request.
        log.Error(err, "Failed to get HelloWorld")
        return ctrl.Result{}, err
    }

    // 2. Define the desired Pod
    pod := &corev1.Pod{
        ObjectMeta: metav1.ObjectMeta{
            Name:      helloWorld.Name + "-pod",
            Namespace: helloWorld.Namespace,
            Labels: map[string]string{
                "app": helloWorld.Name,
            },
        },
        Spec: corev1.PodSpec{
            Containers: []corev1.Container{
                {
                    Name:  "hello-world",
                    Image: "busybox",
                    Command: []string{"sh", "-c", fmt.Sprintf("echo '%s' && sleep 3600", helloWorld.Spec.Message)},
                },
            },
        },
    }

    // 3. Set HelloWorld instance as the owner and controller
    if err := ctrl.SetControllerReference(helloWorld, pod, r.Scheme); err != nil {
        log.Error(err, "Failed to set controller reference")
        return ctrl.Result{}, err
    }

    // 4. Check if the Pod already exists
    found := &corev1.Pod{}
    err = r.Get(ctx, client.ObjectKey{Name: pod.Name, Namespace: pod.Namespace}, found)
    if err != nil && apierrors.IsNotFound(err) {
        log.Info("Creating a new Pod", "Pod.Namespace", pod.Namespace, "Pod.Name", pod.Name)
        err = r.Create(ctx, pod)
        if err != nil {
            log.Error(err, "Failed to create new Pod", "Pod.Namespace", pod.Namespace, "Pod.Name", pod.Name)
            return ctrl.Result{}, err
        }
        // Pod created successfully - return and requeue
        return ctrl.Result{Requeue: true}, nil
    } else if err != nil {
        log.Error(err, "Failed to get Pod")
        return ctrl.Result{}, err
    }

    // 5. Pod already exists - don't requeue
    log.Info("Skip reconcile: Pod already exists", "Pod.Namespace", found.Namespace, "Pod.Name", found.Name)
    return ctrl.Result{}, nil
}

// SetupWithManager sets up the controller with the Manager.
func (r *HelloWorldReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&examplev1alpha1.HelloWorld{}).
        Owns(&corev1.Pod{}).
        Complete(r)
}
```


## 4. 安装 CRD 到 Kubernetes 集群

运行以下命令安装 CRD 到 Kubernetes 集群：

```bash
make install
```

### 5. 运行 Operator

运行以下命令在本地运行 Operator：

```bash
make run
```

### 6. 创建 HelloWorld 资源

创建一个名为 `my-hello-world.yaml` 的文件，内容如下：

```yaml
apiVersion: example.com/v1alpha1
kind: HelloWorld
metadata:
  name: my-hello-world
spec:
  name: my-hello-world
  message: "Hello World from Operator!"
```

使用 `kubectl apply -f my-hello-world.yaml` 创建资源。

### 7. 验证

使用 `kubectl get pods` 命令查看是否创建了名为 `my-hello-world-pod` 的 Pod。 使用 `kubectl logs my-hello-world-pod` 查看 Pod 的日志，确认是否输出了 "Hello World from Operator!"。

## 总结

恭喜你完成了第一个 Operator！  虽然这个 Operator 非常简单，但它展示了 Operator 的基本原理：监听自定义资源，并根据资源的状态来管理 Kubernetes 资源。 在接下来的系列中，我们将深入探讨 Operator 的更多高级特性。

敬请期待下一篇博客！