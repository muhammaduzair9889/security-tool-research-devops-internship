# Falco: Container Runtime Security

## Executive Summary

**Falco** is a CNCF-backed container runtime security tool using eBPF to detect and prevent threats in containerized environments. It's the **specialized choice for Kubernetes-native security**, providing powerful container behavior monitoring without traditional agent installation.

### Key Strengths
- Purpose-built for Kubernetes security
- eBPF-based with minimal overhead
- Real-time threat detection
- CNCF project with strong backing
- No agent installation needed (runs as DaemonSet)

### Overall Score: 4.2/5.0

**Best For**: Kubernetes-focused organizations needing runtime security

---

## Product Overview

Falco is an open-source behavioral security tool developed by Sysdig. It uses eBPF (extended Berkeley Packet Filter) to monitor system calls and detect suspicious behavior in containers and Kubernetes environments without requiring installation of agents inside containers.

### Key Difference from OSSEC/Wazuh
- **Runtime focused**: Detects behavior inside running containers
- **Agentless inside containers**: Uses kernel-level monitoring
- **Kubernetes-native**: Designed for Kubernetes specifically
- **Lightweight**: Minimal performance overhead

---

## Core Features

### 1. **System Call Monitoring**
- Monitor all system calls on host and containers
- Behavior-based threat detection
- Process genealogy tracking
- Real-time alerting

### 2. **Container-Specific Detection**
- Unauthorized process execution
- Container escape attempts
- Sensitive file access
- Network anomalies
- Privilege escalation

### 3. **Kubernetes Integration**
- CRI (Container Runtime Interface) support
- Pod-level context in alerts
- Kubernetes API integration
- RBAC-aware monitoring

### 4. **ML-Based Anomaly Detection**
- Behavioral baseline establishment
- Anomaly scoring
- Learning mode / Enforce mode
- Custom rule creation

---

## Architecture

```
Kubernetes Cluster
├─ Node 1
│  └─ Falco DaemonSet Pod
│     └─ eBPF Module
│        └─ Monitors syscalls for all containers
├─ Node 2
│  └─ Falco DaemonSet Pod
│     └─ eBPF Module
└─ Node 3
   └─ Falco DaemonSet Pod
      └─ eBPF Module

All → Falco Rules Engine → Output to:
      • Slack
      • CloudWatch
      • Datadog
      • Custom webhooks
      • Splunk
```

---

## Deployment Models

### 1. **DaemonSet** (Standard Kubernetes)
- Runs on every node
- Monitors all containers on node
- Native Kubernetes integration
- Helm chart available

### 2. **Sidecar Pattern**
- Runs alongside application pods
- Focused container monitoring
- Pod-level isolation
- Resource-constrained way

### 3. **Host-level Deployment**
- Runs on VM/bare metal
- Kubernetes node monitoring
- Non-Kubernetes container support

---

## Performance Characteristics

### Resource Overhead
- **CPU**: Typically 1-3% overhead
- **Memory**: 100-300 MB per node
- **Network**: Minimal (local processing)

### Scalability
- Per-node deployment (scales horizontally)
- No central bottleneck
- Supports large clusters (100+ nodes)

---

## Cost

### Licensing
- **Free**: Open-source version
- **Falco Cloud**: Commercial SaaS offering

### Infrastructure
- Minimal additional cost
- Shared node resources
- No central storage required

**TCO for 500-node cluster**: $5,000-10,000/year infrastructure only

---

## Strengths

1. **Kubernetes-Native**: Purpose-built for Kubernetes
2. **eBPF Efficiency**: System-level monitoring with minimal overhead
3. **No Container Agent**: Eliminates need for container-level agents
4. **Real-time Detection**: <1 second detection latency
5. **Community-Driven**: Active development, CNCF-backed
6. **Powerful Rules**: Extensive rule library
7. **Cloud-Ready**: Works on AKS, EKS, GKE

---

## Limitations

1. **Kubernetes-Specific**: Not ideal for non-containerized environments
2. **Limited Compliance**: No built-in compliance reporting
3. **Complexity**: Requires understanding of system calls
4. **Rule Definition**: Steep learning curve for custom rules
5. **No Vulnerability Scanning**: Focus is runtime, not inventory
6. **No SIEM**: Alert generation, but limited log correlation
7. **eBPF Requirement**: Requires kernel 4.8+

---

## Best Use Cases

✅ **Excellent For**:
- Kubernetes clusters
- Container runtime security
- Microservices environments
- Organizations using Kubernetes natively

❌ **Not Ideal For**:
- Non-containerized infrastructure
- Organizations needing compliance reporting
- End-point focused security (use OSSEC/Wazuh)
- VMware or traditional VM environments

---

## Scoring

| Criteria | Score |
|----------|-------|
| Functionality (Container) | 5.0 |
| Performance | 5.0 |
| Deployment | 4.0 |
| Cost | 5.0 |
| Integration | 3.5 |
| Support | 3.5 |
| **Overall** | **4.2** |

---

**Recommendation**: **REQUIRED** for Kubernetes environments; supplement with Wazuh for non-container infra.

---

**Document Version**: 1.0 | **Last Updated**: March 5, 2026 | **Task**: DEV-358
