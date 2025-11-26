# CKAD (Certified Kubernetes Application Developer) Study Guide

Welcome! This repository contains comprehensive study materials for the **Certified Kubernetes Application Developer (CKAD)** certification exam. It covers all major sections of the exam curriculum with detailed notes, practical examples, and practice questions.

> 📖 **Course Source:** These notes are based on the excellent **[CKAD Course by Mumshad Mannambeth on Udemy](https://www.udemy.com/course/certified-kubernetes-application-developer/)**, which provides a structured and comprehensive approach to mastering Kubernetes application development.

## 📚 Table of Contents

1. [Repository Overview](#repository-overview)
2. [Exam Sections](#exam-sections)
3. [How to Use This Repository](#how-to-use-this-repository)
4. [Quick References](#quick-references)
5. [Practice Questions](#practice-questions)

---

## Repository Overview

This study guide is organized into **12 main sections**, each focusing on a specific domain of the CKAD certification. Each section contains:
- **Detailed notes** covering concepts and theory
- **YAML configuration examples** for hands-on learning
- **Best practices** and tips for the exam

---

## 📋 Exam Sections

### [00-Cheat-Sheet](./00-Cheat-Sheet/)
**Quick Reference Commands**
- `00-commands.md` - Essential kubectl commands for quick lookup
  - Object creation commands
  - Resource exposure patterns
  - API resource reference
  - Configuration management commands

*Use this section during exam preparation for quick command lookups.*

---

### [01-Core-Concepts](./01-Core-Concepts/)
**Fundamentals of Kubernetes**
- `00-core-concepts-notes.md` - Core Kubernetes concepts
  - PODs basics and lifecycle
  - YAML structure and syntax
  - Replication Controllers
  - ReplicaSets
  - Deployments
  - Namespaces

**Example Files:**
- `01-pod-demo.yml` - Basic Pod configuration
- `02-replication-controller-demo.yml` - Replication Controller example
- `03-replica-set-demo.yml` - ReplicaSet configuration
- `04-deployment-demo.yml` - Deployment example
- `05-namespace-demo.yml` - Namespace creation

*Start here if you're new to Kubernetes. This section covers 13% of the exam.*

---

### [02-Configuration](./02-Configuration/)
**Configuration and Application Deployment**
- `02-configuration-notes.md` - In-depth configuration guide
  - Docker image fundamentals
  - Dockerfile instructions
  - ConfigMaps and Secrets
  - Environment variables
  - Volume mounts
  - Resource limits and requests

**Example Files:**
- `Dockerfile` - Sample Docker configuration

*This section covers 18% of the exam. Understanding ConfigMaps and Secrets is crucial.*

---

### [03-MultiContainer-Pods](./03-MultiContainer-Pods/)
**Advanced Pod Patterns**
- `03-multicontainer-pods-notes.md` - Multi-container Pod design patterns
  - Sidecar containers
  - Init containers
  - Ambassador pattern
  - Adapter pattern

*Covers advanced Pod design patterns (10% of exam).*

---

### [04-Observability](./04-Observability/)
**Monitoring and Debugging**
- `04-observability-notes.md` - Observability in Kubernetes
  - Logging strategies
  - Debugging Pods
  - Health checks (liveness and readiness probes)
  - Metrics and monitoring

*Critical for understanding Pod lifecycle and troubleshooting (18% of exam).*

---

### [05-Pod-Design](./05-Pod-Design/)
**Pod Scheduling and Design**
- `05-pod-design-notes.md` - Pod design patterns and scheduling
  - Labels and Selectors
  - Annotations
  - Node affinity
  - Pod affinity
  - Taints and Tolerations
  - Rolling updates

*Important for production-ready deployments (20% of exam).*

---

### [06-Networking](./06-Networking/)
**Kubernetes Networking**
- `06-networking-notes.md` - Network policies and services
  - Services (ClusterIP, NodePort, LoadBalancer)
  - Ingress
  - Network policies
  - DNS and service discovery

*Essential for inter-Pod communication (13% of exam).*

---

### [07-State-Persistence](./07-State-Persistence/)
**Storage and Persistence**
- `07-state-persistence-notes.md` - Persistent storage in Kubernetes
  - Volumes
  - PersistentVolumes (PV)
  - PersistentVolumeClaims (PVC)
  - Storage Classes
  - StatefulSets

*Covers data persistence and state management.*

---

### [08-Security](./08-Security/)
**Security Best Practices**
- `08-secuirty-notes.md` - Kubernetes security
  - Security contexts
  - Pod security policies
  - RBAC basics
  - Secret management
  - Network policies

*Critical for secure application deployment (14% of exam).*

---

### [09-Helm](./09-Helm/)
**Helm Package Manager**
- `09-helm.md` - Helm basics and package management
  - Helm charts
  - Values and templating
  - Releases and repositories
  - Best practices

*Useful for application packaging and deployment.*

---

### [10-Kustomize](./10-Kustomize/)
**Kustomize Configuration Management**
- `10-kustomize.md` - Kustomize for configuration management
  - Overlays and bases
  - Patches and customizations
  - Multi-environment deployments

*Alternative to Helm for configuration management.*

---

### [11-Practice-Questions](./11-Practice-Questions/)
**Exam-Style Practice Questions**

Multiple practice scenarios with solutions:
- `1.taints-tolerations.md` - Node taints and Pod tolerations
- `2.node-affinity.md` - Node affinity and Pod placement
- `3.rolling-updates.md` - Deployment strategies and updates
- `4.jobs.md` - Kubernetes Jobs and CronJobs
- `5.networkpolicy.md` - Network policy configuration
- `6.persistent-volume-with-pod.md` - Persistent storage scenarios
- `7-sidecar.md` - Sidecar container patterns
- `8.init-container.md` - Init container usage

*These practice questions simulate real exam scenarios. Work through these regularly!*

---

## 🎯 How to Use This Repository

### For Beginners:
1. Start with [01-Core-Concepts](./01-Core-Concepts/) to understand Kubernetes fundamentals
2. Review the YAML examples in each section
3. Progress sequentially through sections 2-8

### For Quick References:
- Use [00-Cheat-Sheet](./00-Cheat-Sheet/) for kubectl commands
- Each section's notes file contains quick reference tables

### For Exam Preparation:
1. **Study Phase:** Read through all notes in order (sections 1-8)
2. **Practice Phase:** Work through [11-Practice-Questions](./11-Practice-Questions/)
3. **Review Phase:** Use the cheat sheet to quickly recall commands
4. **Test Phase:** Create your own scenarios and test them in a local Kubernetes cluster

### Hands-On Practice:
- Use the YAML files provided as templates
- Deploy them to a local Kubernetes cluster (Docker Desktop, Minikube, or Kind)
- Modify configurations and observe the behavior
- Use `kubectl` commands from the cheat sheet

---

## 📖 Quick References

### Common kubectl Commands:
```bash
# Pod operations
kubectl run nginx --image=nginx                    # Create a Pod
kubectl get pods                                  # List all Pods
kubectl describe pod <pod-name>                   # Get Pod details
kubectl logs <pod-name>                           # View Pod logs
kubectl exec -it <pod-name> -- /bin/bash         # Execute command in Pod

# Deployment operations
kubectl create deployment my-app --image=my-app  # Create Deployment
kubectl rollout status deployment/my-app         # Check rollout status
kubectl rollout undo deployment/my-app           # Rollback deployment
kubectl set image deployment/my-app my-app=my-app:v2  # Update image

# Configuration
kubectl create configmap my-config --from-literal=key=value
kubectl create secret generic my-secret --from-literal=key=value

# Scaling
kubectl scale deployment my-app --replicas=3

# Resource inspection
kubectl get all                                   # List all resources
kubectl api-resources                             # List all API resources
```

For more commands, see [00-Cheat-Sheet/00-commands.md](./00-Cheat-Sheet/00-commands.md)

---

## 🧪 Practice Questions Summary

| Topic | File | Level |
|-------|------|-------|
| Taints & Tolerations | `1.taints-tolerations.md` | Beginner |
| Node Affinity | `2.node-affinity.md` | Intermediate |
| Rolling Updates | `3.rolling-updates.md` | Intermediate |
| Jobs & CronJobs | `4.jobs.md` | Intermediate |
| Network Policy | `5.networkpolicy.md` | Advanced |
| Persistent Volumes | `6.persistent-volume-with-pod.md` | Advanced |
| Sidecar Pattern | `7-sidecar.md` | Advanced |
| Init Containers | `8.init-container.md` | Advanced |

---

## 🎓 CKAD Exam Information

**Exam Duration:** 2 hours  
**Passing Score:** 66%  
**Format:** Practical hands-on tasks in a real Kubernetes cluster

### Exam Domains & Weights:
- Application Design and Build - 20%
- Application Deployment - 20%
- Application Observability and Maintenace - 15%
- Application Environment, Configuration and Security - 25%
- Service and Networking - 20%

---

## 💡 Study Tips
1. **Use the Docs:** Familiarize yourself with the official Kubernetes documentation
2. **Time Management:** Practice completing tasks under time pressure
3. **Use kubectl Aliases:** Create shortcuts like `k` for `kubectl`
4. **Dry Run:** Use `--dry-run=client -o yaml` to generate manifests

---

## 📚 Additional Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [CKAD Exam Curriculum](https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/)
- [Kubernetes Official Tutorials](https://kubernetes.io/docs/tutorials/)
- [kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)

---

## 📝 Notes

- These study materials are compiled from **Mumshad Mannambeth's CKAD Udemy course**
- All YAML examples should be functional, if any errors please report.
- Practice questions include solutions with explanations
- Keep this repository updated with new learnings and practice scenarios

---

## 🤝 Contributing

Feel free to add more practice questions, examples, or clarifications to help future learners.

---

**Good luck with your CKAD certification journey!** 🚀

Last Updated: November 2025