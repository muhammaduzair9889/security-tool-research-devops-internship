# Tools Overview: Research Scope & Selection

## Overview

This document provides a high-level overview of all tools selected for this research, organized by security domain. Each tool represents a significant player in its category, offering distinct architectural approaches and feature sets.

## Domain 1: Intrusion Detection Systems (IDS)

### Selection Criteria
Tools were selected based on:
- Relevance to DevOps/cloud-native environments
- Maturity and proven deployments
- Open-source vs commercial representation
- Different architectural approaches

### Selected Tools

#### 1. OSSEC (Open Source Security)

**Classification**: Open-source Host-based IDS (HIDS)

**Key Characteristics**:
- Lightweight, agent-based monitoring
- Focus on file integrity monitoring (FIM)
- Minimal resource requirements
- Community-driven development

**Primary Use Case**: Cost-sensitive deployments, legacy system monitoring

**Licensing**: Free (open-source)

---

#### 2. Wazuh (Enterprise IDS/SIEM)

**Classification**: Open-source Enterprise IDS with SIEM capabilities

**Key Characteristics**:
- Extended OSSEC with advanced features
- Cloud-native architecture
- Extensive AWS/cloud integration
- Commercial support available

**Primary Use Case**: Enterprise-scale cloud-native deployments

**Licensing**: Free (open-source) or commercial support

---

#### 3. Falco (Container Runtime Security)

**Classification**: Open-source Container Runtime Security

**Key Characteristics**:
- eBPF-based system call monitoring
- Kubernetes-native
- Real-time threat detection
- CNCF-incubating project

**Primary Use Case**: Kubernetes cluster security, container threat detection

**Licensing**: Free (open-source)

---

## Domain 2: Vulnerability Scanners

### Selection Criteria
Tools were selected for:
- Container image scanning capabilities
- Integration with CI/CD pipelines
- Vulnerability database currency
- Performance characteristics

### Selected Tools

#### 1. OpenVAS (Open Vulnerability Assessment System)

**Classification**: Comprehensive vulnerability scanning framework

**Key Characteristics**:
- Full-featured vulnerability scanner
- Network and host-based scanning
- Extensive reporting
- Mature, established tool

**Primary Use Case**: Comprehensive vulnerability assessments

**Licensing**: Free (open-source, limited DB) or commercial

---

#### 2. Trivy (Trivial Vulnerability Scanner)

**Classification**: Fast container image vulnerability scanner

**Key Characteristics**:
- Designed for container images
- Very fast (< 1 minute per image)
- Minimal false positives
- CI/CD pipeline optimized

**Primary Use Case**: DevOps pipeline vulnerability scanning

**Licensing**: Free (open-source)

---

#### 3. Grype (SBoM-focused Vulnerability Scanner)

**Classification**: Dependency vulnerability scanner with SBoM generation

**Key Characteristics**:
- Focus on application dependencies
- Software Bill of Materials (SBoM) generation
- Supply chain security focus
- Syft integration for inventory

**Primary Use Case**: Application-level vulnerability management

**Licensing**: Free (open-source)

---

## Domain 3: Centralized Logging & SIEM

### Selection Criteria
Tools were selected for:
- Log aggregation capabilities
- Scalability for large log volumes
- Analytics and correlation features
- Integration ecosystem

### Selected Tools

#### 1. ELK Stack (Elasticsearch-Logstash-Kibana)

**Classification**: Open-source Log management and analytics platform

**Key Characteristics**:
- Mature, widely deployed
- Powerful search and aggregation
- Flexible data pipeline
- Large community ecosystem

**Primary Use Case**: Enterprise log aggregation and analysis

**Licensing**: Free (open-source, with commercial options)

---

#### 2. Grafana Loki (Lightweight Log Aggregation)

**Classification**: Kubernetes-optimized log aggregation

**Key Characteristics**:
- Designed for log volumes (not sampling)
- Superior cost efficiency
- Native Kubernetes integration
- Part of Grafana stack

**Primary Use Case**: Kubernetes-native log aggregation

**Licensing**: Free (open-source)

---

#### 3. Graylog (Unified Log Management)

**Classification**: Full-featured log management platform

**Key Characteristics**:
- Stream processing capabilities
- Content packs for quick setup
- Pipeline-based processing
- Full-text search

**Primary Use Case**: Unified log management with analytics

**Licensing**: Free (open-source) or commercial

---

## Evaluation Matrix Summary

| Domain | Tool | Type | Scale | Cost | Best For |
|--------|------|------|-------|------|----------|
| **IDS** | OSSEC | Host-based | Small-Medium | Free | Cost-sensitive, legacy |
| | Wazuh | Enterprise | Medium-Large | Free/Commercial | Cloud-native enterprise |
| | Falco | Runtime | Medium-Large | Free | Kubernetes security |
| **Scanners** | OpenVAS | Network | On-demand | Free/Commercial | Comprehensive scanning |
| | Trivy | Image | CI/CD | Free | DevOps pipeline |
| | Grype | Dependencies | Build-time | Free | Supply chain security |
| **Logging** | ELK Stack | Aggregation | Large | Free/Commercial | Enterprise scale |
| | Loki | Log streaming | Kubernetes | Free | Kubernetes native |
| | Graylog | Unified | Medium | Free/Commercial | Log analytics |

## Document Structure by Tool

### For Each Tool Analysis, See:

**IDS Tools**:
- [OSSEC Analysis](./04-ids-analysis/ossec.md)
- [Wazuh Analysis](./04-ids-analysis/wazuh.md)
- [Falco Analysis](./04-ids-analysis/falco.md)
- [IDS Summary & Comparison](./04-ids-analysis/ids-summary.md)

**Vulnerability Scanners**:
- [OpenVAS Analysis](./05-vulnerability-scanners/openvas.md)
- [Trivy Analysis](./05-vulnerability-scanners/trivy.md)
- [Grype Analysis](./05-vulnerability-scanners/grype.md)
- [Vulnerability Scanner Summary](./05-vulnerability-scanners/vuln-scanner-summary.md)

**Logging & SIEM**:
- [ELK Stack Analysis](./06-centralized-logging/elk-stack.md)
- [Grafana Loki Analysis](./06-centralized-logging/loki.md)
- [Graylog Analysis](./06-centralized-logging/graylog.md)
- [Logging Tools Summary](./06-centralized-logging/logging-summary.md)

## Tools Not Included (Rationale)

### Vulnerability Scanners Not Included
- **Nessus**: Commercial tool; cost-benefit analysis done in cost comparison
- **Qualys**: Cloud-only SaaS; different deployment model
- **Acunetix**: Web-focused; different use case

### Logging Tools Not Included
- **Splunk**: Commercial; cost analysis included
- **Datadog**: SaaS/managed; cost analysis included
- **New Relic**: APM-focused; different primary use case

### IDS Tools Not Included
- **Suricata**: Network IDS (different focus from host-based)
- **Zeek**: Network traffic analysis (different focus)
- **AIDE**: File integrity only (limited scope)

## Key Distinctions

### Architecture Approaches

**Agent-based** (OSSEC, Wazuh, Falco)
- Software runs on monitored systems
- Real-time processing on endpoint
- Lower false positive rates
- More operational overhead

**Agentless** (OpenVAS scanning, Loki collection)
- Centralized point of presence
- Lower endpoint overhead
- Easier multi-environment deployment
- Potential network overhead

### Deployment Models

**Host/Instance-level** (OSSEC, Falco agents)
- Installed and configured per host
- Enable compliance with industry standards
- Host-centric security model

**Cluster-level** (Falco with Kubernetes)
- Deployed as daemon set or sidecar
- System-wide monitoring
- Cloud-native principles

**Centralized** (ELK, Graylog, Wazuh server)
- Central collection and analysis
- Unified visibility
- Single point of management

## Feature Grouping

### By Primary Function

**Detection-focused**: OSSEC, Wazuh, Falco
- Emphasis on threat/anomaly detection
- Real-time alerting
- Investigation capabilities

**Scanning-focused**: OpenVAS, Trivy, Grype
- Vulnerability identification
- Compliance checking
- Report generation

**Aggregation-focused**: ELK, Loki, Graylog
- Log collection and storage
- Search and analysis
- Correlation and alerting

## Research Progression Path

For best understanding, read documents in this order:

1. **Quick Assessment**
   - This document (tools overview)
   - Individual tool summaries

2. **Deep Dive Analysis**
   - Individual tool detail pages
   - Domain-specific comparison documents

3. **Decision Making**
   - Cost analysis comparisons
   - Implementation complexity
   - Final recommendations

4. **Implementation**
   - Suggested architecture
   - Implementation roadmap
   - Risk and mitigation strategies
