---
title: Cloud Edition
---
## Overview

QBO Cloud Edition enables users to leverage the power of **QBO Kubernetes Engine (QKE)** and **QBO Container Engine (QCE)** for managing Kubernetes clusters and compute workloads with unmatched efficiency and speed. Built to deliver bare-metal performance within a cloud-native experience, QBO Cloud bridges the gap between high-performance computing and the convenience of cloud infrastructure.

With QBO, users gain:

* Direct access to **GPU**, **CPU**, and **Disk** resources
* Lightning-fast operations for cluster and instance lifecycle management
* Full control via **Web UI**, **CLI**, or the **QBO AsyncAPI**

Whether you're running AI/ML models, database workloads, or microservices, QBO Cloud delivers the performance of dedicated hardware combined with the elasticity of cloud operations.

## Key Benefits

* **Bare-metal speed** with zero virtualization overhead
* **Elastic scaling** of Kubernetes clusters and standalone containers
* **Unified interface** to manage workloads: GUI, CLI, or AsyncAPI
* **Seamless GPU integration** for compute-heavy workloads
* **Real-time observability** powered by QBO's event-driven architecture

## Configuration Options

Access to QBO Cloud can be configured using two main interfaces:

### Web Interface

- Authenticate using **OAuth2 via Google**
- Access all QBO resources and operations through a browser
- Ideal for interactive workflows, administrators, and real-time monitoring

For a full walkthrough of the web interface, refer to the [Web Interface Guide](web).


### CLI Access

* Authenticate using one of two methods:

  1. **Temporary Token**: Retrieved via browser session (OAuth2)
  2. **Service Account**: Persistent credentials stored locally

For more details on setting up CLI access, refer to the [Command Line Interface guide](cli).

---

QBO Cloud offers a production-grade, high-performance compute environment that is ready to deploy workloads in seconds. Whether you need to launch a full Kubernetes cluster or spin up individual compute containers, QBO gives you the tools to do so with minimal latency and maximum control.
