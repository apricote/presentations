# Kubebuilder

---

### Content

- CRDs
- Controller / Operator
- Kubebuilder
- Live Coding
- Real Life Use Cases

---

## CRD's

- introduced in Kubernetes 1.7
- replacing "third party ressources"
- enables users to create own Kubernetes Objects
- can be used like any other object

--

<!--### Costum Resource

- allows to add objects to Kubernetes API
- allows to create own API

-- -->

## Resources and Kinds

> "A resource is an endpoint in the Kubernetes API that stores a collection of API objects of a certain kind."

- [Costum Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

--

## Kinds

- API types
- called "Kinds"
- e.g. Pod, Deployment etc.

--

## Resources

- use of a "Kind"
- Pod resource holds Pod Objects

---

## Controller

> A **control loop** that watches the state [...] and makes changes attempting to move the **current state towards the desired state**.

- [Kubernetes Glossary](https://kubernetes.io/docs/reference/glossary/?fundamental=true#term-controller)

--

## Controller

- integrated in `kube-apiserver` or `kube-controller-manager`
- watch builtin k8s objects (`core`, `apps`, `batch`)
- logic ontop of data
- create sub-resources

--

## Controller

### deployment controller

- watches `Deployments`
- creates and deletes `ReplicaSets`
- scales `ReplicaSets` for rolling deployments

--

## Controller

### kubelet

- watches `PodSpecs`
- starts/stops containers

--

## Operator

- term coined by CoreOS
- same as controller but for CRDs
- integrate operational knowledge
- provide usability features
  - updates
  - scaling
  - backups
  - integration with k8s

---

## Kubebuilder

> "Kubebuilder is a framework for building Kubernetes APIs using custom resource definitions (CRDs)."

- [kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)

--

## Kubebuilder

- First release in March 2018
- Version 2.0.0 five days ago

--

## Kubebuilder

### Motivation

- need to create a lot of files
- need to create much boilerplate code

--

## Kubebuilder

### What Kubebuilder is doing..

- generates the whole project
- CRD's, RBAC settings, tests etc.
- so you can focus on your use case

---

## Live Coding

Check out the source at  
https://github.com/apricote/kubebuilder-poke-sync

---

## Real Life Use Cases

### Databases

- scaling
- configuration
- disaster recovery
- e.g. [zalando/postgres-operator](https://github.com/zalando/postgres-operator)
- e.g. [mongodb-enterprise-kubernetes](https://github.com/mongodb/mongodb-enterprise-kubernetes)

--

## Real Life Use Cases

### Provisioning external resources

- call external APIs
- expose external state in cluster
- e.g. [jetstack/cert-manager](https://github.com/jetstack/cert-manager)
- e.g. [awslabs/aws-service-operator](https://github.com/awslabs/aws-service-operator)

--

## Real Life Use Cases

### Application Lifecycle Managment

- deployment
- supervised updates
- health checking + alerts
- Helm Chart++
- e.g. [gitlab-operator](https://gitlab.com/charts/components/gitlab-operator)
- e.g. [Tenant Operator](https://blog.kolide.com/using-a-kubernetes-operator-to-manage-tenancy-in-a-b2b-saas-app-250f1c9416ce)
