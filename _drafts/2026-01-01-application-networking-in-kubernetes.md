---
layout: post
title:  "Network Security in Kubernetes with Cilium: Limiting the Blast Radius"
date:   2026-01-01 15:43:20 +0100
categories: kubernetes security
---
# Introduction: Kubernetes Adoption and the Security Reality

**Key points:**

* Kubernetes has become a widely adopted platform for deploying and operating applications on self-managed infrastructure.
* It enables scalability, resilience, and operational consistency across environments.
* As adoption grows, Kubernetes clusters increasingly become high-value targets.
* Security must be treated as a first-class concern, not an afterthought.


# Why Network Security Matters in Kubernetes

**Key points:**

* Kubernetes is a distributed system with many components communicating over the network.
* By default, pods can often communicate freely within a cluster.
* Overly permissive networking increases the attack surface.
* Network security is about **restricting access to only what is necessary**.

# Lessons from Recent Exploits (React2Shell)

**Key points:**

* The react2shell exploit demonstrated how a frontend vulnerability can lead to remote code execution.
* Such vulnerabilities are difficult to fully prevent at the application layer alone.
* Once an attacker gains code execution, lateral movement becomes the real danger.
* Kubernetes networking controls can dramatically reduce the impact of such breaches.


# The “Assume Breach” Security Model

**Key points:**

* Modern security assumes that attackers may eventually gain a foothold.
* The goal shifts from “prevent all attacks” to “limit the blast radius.”
* Network isolation is a key control for containing compromised workloads.
* Kubernetes provides primitives to enforce this isolation.

# Kubernetes Network Policies: The Foundation

**Key points:**

* NetworkPolicies define which pods can communicate with each other.
* They enforce **default-deny** behavior when properly configured.
* Policies are namespace-aware and label-based.
* Without enforcement by a capable CNI, NetworkPolicies are ineffective.

# Why Cilium for Kubernetes Networking

**Key points:**

* Cilium is a CNI that uses eBPF for high-performance networking and security.
* It provides full Kubernetes NetworkPolicy support and extensions.
* Policies can be enforced at Layer 3, 4, and 7.
* Cilium enables visibility and control without significant performance penalties.


# Restricting East-West Traffic with Cilium

**Key points:**

* Most attacks rely on moving laterally within the cluster.
* Cilium allows fine-grained pod-to-pod access control.
* Only explicitly allowed services can communicate.
* This prevents compromised pods from scanning or attacking others.

# Applying Least Privilege Networking

**Key points:**

* Each workload should only access what it strictly needs.
* Backend services should not be reachable from unrelated pods.
* Databases should only accept traffic from authorized applications.
* Cilium policies make least-privilege networking practical at scale.


# Limiting Damage After Remote Code Execution

**Key points:**

* If an attacker gains RCE on a pod, network rules define their reach.
* Proper policies can block access to:

  * Internal APIs
  * Databases
  * Metadata services
  * Control plane components
* The attacker is effectively trapped inside a small, controlled zone.

# Observability and Auditability with Cilium

**Key points:**

* Cilium provides visibility into allowed and denied connections.
* Flow logs help identify unexpected communication patterns.
* This aids in detection, forensics, and policy refinement.
* Security teams gain confidence in enforcement effectiveness.


# Conclusion: Reducing Risk Through Network Isolation

**Key points:**

* Kubernetes enables powerful, scalable deployments—but also complex attack surfaces.
* Vulnerabilities like react2shell show that breaches are sometimes unavoidable.
* Strong network security significantly reduces the impact of such breaches.
* Using Cilium to enforce least-privilege networking is a practical, effective defense strategy.


# My stuff

Hopefully at this point you are convinced that network policies are really useful and should definetly be used in production. Thankfully cilium makes it really easy for us to specify what our pods can talk to (and implictly what they are not allowed to talk to, namely everything else).

You can quickly test it yourself following this great tutorial on the cilium docs:

https://docs.cilium.io/en/stable/security/dns/

All you need to do is get a local minikube cluster running (5 minutes) and apply a couple of CiliumNetworkPolicies to the cluster (around another 10 minutes) and you will get a basic understanding how things work. 

## Question

How does this port 80 thing work? How is it possible to filter on fqdn when you don't validate the HTTPS cert. Shouldnt it be necessary to only allow HTTPS traffic?