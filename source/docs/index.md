---
title: Overview
---

### What is [qbo](def#Glossary)

QBO is an AsyncAPI designed to streamline the deployment of [Kind](https://kind.sigs.k8s.io/) images using Docker containers as Kubernetes nodes. It empowers users to effortlessly deploy, manage, and scale containerized applications using Kubernetes, eliminating the need for intricate infrastructure management.

With Kubernetes Engine (QKE), users can unlock the full potential of [metal performance](https://en.wikipedia.org/wiki/Bare-metal_server), particularly beneficial for compute-intensive workloads demanding high performance and minimal latency, such as databases and AI/ML models.

QBO offers **QBO Kubernetes Engine Community Edition (QKE CE)** at no cost, compatible with Linux or Windows (WSL2), enabling local execution. Additionally, QBO provides a fully managed cloud solution through **QBO Kubernetes Engine Cloud (QKE Cloud)**.

### Technology

#### Designed for AI and ML
---
Delivering bare metal with the flexibility of the cloud, QBO excels in handling compute-intensive workloads that require high performance and low latency — such as databases, AI/ML models, and real-time applications. If your user experience and business success hinge on performance, leveraging bare metal with QBO technology should be a strategic consideration.

#### Pure containers
---
QBO operates as an AsyncAPI that manages Kubernetes-in-Docker (KinD) deployments within the QBO Kubernetes Engine (QKE). In this setup, Kubernetes nodes are redefined as Docker containers. Containerd runs inside Docker, allowing an entire Kubernetes infrastructure to function as a self-contained process directly on the hardware. Similarly, QBO supports Docker-in-Docker (DinD) deployments for compute instances, eliminating the need for traditional virtualization.

#### Metal Performance
---
QBO delivers direct access to hardware resources — CPU, memory, and disk — providing an immediate performance boost. Operations like cluster creation, deletion, and scaling are faster than with virtual machines. QBO maintains metal efficiency while offering container-based resource isolation.

#### Native Kernel Functions
---
By managing a single Linux kernel across Kubernetes nodes, QBO offers efficient network, storage, and security operations through a unified interface. It leverages native Linux kernel features including IPVS, netfilter, iproute2, and eBPF. Security and observability are enhanced by eBPF instrumentation on the single host kernel.

#### Unified AsyncAPI
---
QBO acts as a high-performance proxy AsyncAPI for managing Kubernetes clusters and cloud components. It provides real-time status reporting of containers, pods, processes, and threads using JSON-formatted data and command-based interaction.

#### Real-time Communication System
---
To handle dynamic operations and diverse data sources, QBO uses websockets for real-time state updates. 'Mirrors' allow subscribers to receive targeted updates, whether from Kubernetes, Docker, system threads, or authentication services. For example, pod creation or process execution becomes immediately visible through the subscribed mirror.

## Features

  <style>
.article-content table th {
      padding: 10px;
      width: 100%;
    }
  </style>
  
| Feature                       | CLOUD | CE  | Notes |
| ----------------------------- | ----- | --- | ----- |
| Multi-cluster management      | X     | X   |       |
| Cluster scaling               | X     | X   |       |
| In-memory cache               | X     |     |       |
| SSL                           | X     |     |       |
| Google OAuth2                 | X     |     |       |
| Cloud Load Balancer           | X     |     |       |
| IP Network management         | X     |     |       |
| Service Accounts              | X     |     |       |
| Kubeconfig management         | X     | X   |       |
| Kubernetes API Integration    | X     |     |       |
| CNI kindnet                   | X     | X   |       |
| Persistent Storage            | X     | X   |       |
| Gitlab Registry               |       | X   |       |
| Docker Hub Registry           | X     | X   |       |
| Kind Kubernetes image support | X     | X   |       |
| Custom image support          | X     | X   |       |
| Websockets API                | X     | X   |       |
| CLI                           | X     | X   |       |
| Web interface                 | X     | X   |       |
| Web terminal                  | X     | X   |       |
| Kubernetes Engine (QKE)       | X     | X   |       |
| Container Engine (QCE)        | X     |     |       |