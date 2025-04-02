---
title: "Getting Started with Operator Development Series (Part 1): Hello World!"
description: "Understand the basic concepts of Kubernetes Operators and learn how to create a simple `Hello World` Operator using Kubebuilder. This tutorial details the steps for environment preparation, project initialization, CRD definition, Reconcile logic implementation, and Operator deployment and validation, helping you quickly get started with Operator development."
slug: operator
date: 2025-04-01 08:00:00+0000
image: cover.png
categories: [ DevOps ]
tags: [ Kubernetes, Operator ]
weight: 1
---

## Background

Our company recently decided to migrate our products to a Kubernetes environment. To better manage and automate our applications, we chose to use Kubernetes Operators. This blog series will document our process of learning and developing Operators, and we hope it can help others get started with Operator development as well.

## Target Audience

*   Developers and operations personnel with some understanding of Kubernetes.
*   Individuals who want to use Operators to automate application management.
*   People with a basic understanding of the Go language.

## Prerequisites

Before you begin, you'll need the following environment set up:

*   **Go Language Environment (version 1.23 or higher):** Operators are typically developed in Go. You need to install the Go environment, preferably version 1.21 or later (the example uses features compatible with 1.21+, but 1.23+ is recommended per the Kubebuilder requirement often seen). You can download the installation package from [https://go.dev/dl/](https://go.dev/dl/). After installation, please configure your `GOPATH` and `PATH` environment variables correctly.

*   **Kubernetes Cluster:** You need an accessible Kubernetes cluster to deploy and test the Operator. You can use tools like Minikube, Kind, or any other Kubernetes distribution.

*   **kubectl Command-Line Tool:** `kubectl` is the Kubernetes command-line tool used to interact with the cluster. Ensure you have `kubectl` installed, configured, and can connect to your Kubernetes cluster.

*   **Kubebuilder (version 3.0 or higher):** Kubebuilder is a framework for rapidly building Kubernetes Operators. Using Kubebuilder simplifies the development process and generates necessary boilerplate code. You can install Kubebuilder using the following commands:

```bash
# Adjust GOOS/GOARCH if needed, check Kubebuilder docs for specific versions
OS=$(go env GOOS)
ARCH=$(go env GOARCH)
# Verify KubeBuilder version compatibility with your Go and K8s versions
# Example for a specific version, check latest releases on Kubebuilder site
# curl -L -o kubebuilder https://github.com/kubernetes-sigs/kubebuilder/releases/download/v3.x.y/kubebuilder_${OS}_${ARCH}

# Or, get the latest (use with caution, check compatibility)
curl -L -o kubebuilder "https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)"

# Make it executable and move to a bin directory
chmod +x kubebuilder
sudo mv kubebuilder /usr/local/bin/ # Or another directory in your PATH, like $HOME/go/bin
```

    > Ensure the directory where you place `kubebuilder` (e.g., `/usr/local/bin` or `$HOME/go/bin`) is in your `PATH` environment variable. You can run `kubebuilder version` to verify the installation.

*   **Docker (Optional):** You need Docker installed if you intend to build Docker images for your Operator.

> My development environment is macOS (arm64) with OrbStack.

## What is an Operator?

Simply put, an Operator is an extension to Kubernetes that uses Custom Resources (CRs) to automate the management of applications. Operators allow us to manage complex applications (like databases, message queues, etc.) in the same way we manage built-in Kubernetes resources.

## Why Choose Operators?

Operators provide a declarative way to manage the lifecycle of applications, including deployment, upgrades, backups, and recovery. They can simplify operational workflows, increase automation, and ensure that the application state matches the desired configuration.

## Our First Operator: Hello World

This Operator will watch for a Custom Resource named `HelloWorld` and create a Pod in Kubernetes. This Pod will run a simple application that prints "Hello World".

### 1. Initialize the Kubebuilder Project

First, we need to create a new project using Kubebuilder. Create a new directory within your `GOPATH` (or any preferred location if using Go modules outside GOPATH), for example, `hello-world-operator`. Then, navigate into that directory and run the following command:

```bash
# Replace example.com and repo path with your own details if desired
kubebuilder init --domain example.com --repo github.com/your-user/hello-world-operator
```

> **Note:** The original used `--domain infini.cloud --repo github.com/infinilabs/hello-world-operator`. Remember to replace `github.com/your-user/hello-world-operator` with your actual repository path if you plan to host it. The domain is used for the API group.

This command initializes a new Kubebuilder project, generating several necessary files and directories.

### 2. Create the Custom Resource Definition (CRD)

Next, we define the structure for our `HelloWorld` resource. Run the following command:

```bash
kubebuilder create api --group example --version v1alpha1 --kind HelloWorld
```

This command creates a new API definition, generating files including `api/v1alpha1/helloworld_types.go` (for the type definition) and `controllers/helloworld_controller.go` (for the reconciliation logic).

Edit the `api/v1alpha1/helloworld_types.go` file. Modify the `HelloWorldSpec` definition to include `name` and `message` fields:

```go
// HelloWorldSpec defines the desired state of HelloWorld
type HelloWorldSpec struct {
    // INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
    // Important: Run "make" to regenerate code after modifying this file

    // Name specifies the name for resources created by this HelloWorld resource.
    // +optional
    Name string `json:"name,omitempty"`

    // Message is the message to be printed by the pod.
    Message string `json:"message"` // Made mandatory for simplicity in example
}

// HelloWorldStatus defines the observed state of HelloWorld
type HelloWorldStatus struct {
	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// PodName is the name of the Pod created by the HelloWorld resource.
	// +optional
	PodName string `json:"podName,omitempty"`
}

```
> **Note:** I've made `Message` mandatory (`json:"message"`) as the controller logic uses it directly. Added `+optional` markers and a basic `Status` field as good practice, although the controller logic below doesn't update the status yet. Run `make manifests` after editing this file.

### 3. Implement the Reconcile Logic

Edit the `controllers/helloworld_controller.go` file. Implement the `Reconcile` function to create a Pod running a `busybox` image that echoes the `message` defined in the `HelloWorld` resource.

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

	examplev1alpha1 "github.com/your-user/hello-world-operator/api/v1alpha1" // !! Update this import path !!
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
func (r *HelloWorldReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	log := log.FromContext(ctx)
	log.Info("Reconciling HelloWorld")

	// 1. Fetch the HelloWorld instance
	helloWorld := &examplev1alpha1.HelloWorld{}
	if err := r.Get(ctx, req.NamespacedName, helloWorld); err != nil {
		if apierrors.IsNotFound(err) {
			// Object not found, likely deleted. Return without error.
			log.Info("HelloWorld resource not found. Ignoring since object must be deleted")
			return ctrl.Result{}, nil
		}
		// Error reading the object - requeue the request.
		log.Error(err, "Failed to get HelloWorld")
		return ctrl.Result{}, err
	}

	// Use spec.Name if provided, otherwise default to CR name for the Pod name prefix
	podNamePrefix := helloWorld.Spec.Name
	if podNamePrefix == "" {
		podNamePrefix = helloWorld.Name
	}
	podName := podNamePrefix + "-pod"

	// 2. Define the desired Pod
	desiredPod := &corev1.Pod{
		ObjectMeta: metav1.ObjectMeta{
			Name:      podName,
			Namespace: helloWorld.Namespace,
			Labels: map[string]string{
				"app": helloWorld.Name, // Label with the CR name for easier selection
			},
		},
		Spec: corev1.PodSpec{
			Containers: []corev1.Container{
				{
					Name:  "hello-world-container", // More descriptive container name
					Image: "busybox",
					// Use echo and sleep; echo prints the message from the spec
					Command: []string{"/bin/sh", "-c", fmt.Sprintf("echo 'Message from %s: %s'; sleep 3600", helloWorld.Name, helloWorld.Spec.Message)},
				},
			},
			RestartPolicy: corev1.RestartPolicyOnFailure, // Example policy
		},
	}

	// 3. Set HelloWorld instance as the owner and controller of the Pod
	// This ensures the Pod is garbage collected when the HelloWorld CR is deleted
	if err := ctrl.SetControllerReference(helloWorld, desiredPod, r.Scheme); err != nil {
		log.Error(err, "Failed to set controller reference for Pod")
		return ctrl.Result{}, err
	}

	// 4. Check if the Pod already exists
	foundPod := &corev1.Pod{}
	err := r.Get(ctx, client.ObjectKey{Name: desiredPod.Name, Namespace: desiredPod.Namespace}, foundPod)

	// If Pod does not exist, create it
	if err != nil && apierrors.IsNotFound(err) {
		log.Info("Creating a new Pod", "Pod.Namespace", desiredPod.Namespace, "Pod.Name", desiredPod.Name)
		err = r.Create(ctx, desiredPod)
		if err != nil {
			log.Error(err, "Failed to create new Pod", "Pod.Namespace", desiredPod.Namespace, "Pod.Name", desiredPod.Name)
			return ctrl.Result{}, err
		}
		// Pod created successfully - don't requeue immediately, wait for watch event
		log.Info("Pod created successfully")
		return ctrl.Result{}, nil // Usually better to rely on watches than requeueing immediately
	} else if err != nil {
		// Other error trying to get the Pod
		log.Error(err, "Failed to get Pod")
		return ctrl.Result{}, err
	}

	// 5. Pod already exists - potentially check/update if needed (skipped for this simple example)
	log.Info("Skip reconcile: Pod already exists", "Pod.Namespace", foundPod.Namespace, "Pod.Name", foundPod.Name)
	// If we needed to update the pod based on CR changes, we would do it here.
	// For this example, we do nothing if the Pod exists.

	// (Optional) Update HelloWorld status - Good practice but omitted for brevity here
	// helloWorld.Status.PodName = foundPod.Name
	// if err := r.Status().Update(ctx, helloWorld); err != nil {
	//     log.Error(err, "Failed to update HelloWorld status")
	//     return ctrl.Result{}, err
	// }

	return ctrl.Result{}, nil
}

// SetupWithManager sets up the controller with the Manager.
func (r *HelloWorldReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&examplev1alpha1.HelloWorld{}). // Watch for HelloWorld resources
		Owns(&corev1.Pod{}).                // Watch for Pods owned by HelloWorld
		Complete(r)
}

```
> **Important:** Replace `github.com/your-user/hello-world-operator` in the import path with the actual path you used in `kubebuilder init --repo`. Run `make manifests generate` after editing the controller file to update generated code and manifests.

### 4. Install the CRD into the Kubernetes Cluster

Run the following command to install the Custom Resource Definition into your cluster:

```bash
make install
```

### 5. Run the Operator

Run the following command to run the Operator locally (it will connect to your configured Kubernetes cluster):

```bash
make run
```
> Keep this terminal running.

### 6. Create the HelloWorld Resource

In a **new terminal**, create a file named `my-hello-world.yaml` with the following content:

```yaml
apiVersion: example.com/v1alpha1 # Ensure group matches your --group flag
kind: HelloWorld
metadata:
  name: my-hello-world-sample # Changed name slightly to avoid conflict with potential Pod name logic
  namespace: default # Specify namespace or use kubectl default
spec:
  name: my-hello # Name used for the Pod prefix
  message: "Hello World from my first Operator!"
```

Apply this resource using `kubectl`:

```bash
kubectl apply -f my-hello-world.yaml
```

### 7. Verify

Check if the Pod was created (using the `spec.name` field + `-pod` suffix as defined in the controller):

```bash
kubectl get pods -n default
```
You should see a Pod named `my-hello-pod` (or similar based on your spec.name).

Check the logs of the Pod to confirm it printed the message:

```bash
# Replace my-hello-pod with the actual pod name from 'kubectl get pods'
kubectl logs my-hello-pod -n default
```

You should see output similar to: `Message from my-hello-world-sample: Hello World from my first Operator!`

## Conclusion

Congratulations on creating your first Operator! While this example is simple, it demonstrates the fundamental principle of Operators: watching Custom Resources and managing Kubernetes resources based on their desired state. In the upcoming parts of this series, we will delve into more advanced Operator features.

Stay tuned for the next post!