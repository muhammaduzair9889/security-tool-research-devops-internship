# Suggested Security Architecture

## Architecture Overview

This document presents a reference architecture for deploying the recommended security stack (Wazuh + Trivy + ELK) in enterprise environments, with specific guidance for cloud-native, hybrid, and on-premises deployments.

---

## Logical Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                    DATA SOURCES LAYER                              │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐               │
│  │   Hosts     │  │ Kubernetes  │  │ Cloud        │               │
│  │ • Linux     │  │ • DaemonSet │  │ • CloudTrail │               │
│  │ • Windows   │  │ • Pod logs  │  │ • Logs       │               │
│  │ • macOS     │  │ • API audit │  │ • Metrics    │               │
│  └─────┬───────┘  └──────┬──────┘  └──────┬───────┘               │
│        │                  │                │                       │
└────────┼──────────────────┼────────────────┼──────────────────────┘
         │                  │                │
         │                  ▼                │
         │         ┌─────────────────────┐  │
         │         │  Kubernetes Cluster │  │
         │         │  • Falco DaemonSet  │  │
         │         │  • Filebeat sidecar │  │
         │         └────────┬────────────┘  │
         │                  │                │
         └──────────────────┼────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│              COLLECTION & FORWARDING LAYER                       │
├───────────────────────────────────────────────────────────────┐
│                                                               │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Data Collectors (Agents)                        │        │
│  │  • Wazuh Agents (700 KB each)                    │        │
│  │  • Filebeat (lightweight log shipping)           │        │
│  │  • Metricbeat (system metrics)                   │        │
│  │  • Auditbeat (audit logs)                        │        │
│  └────────┬───────────────────────────────────────┘        │
│           │                                                 │
│           │ (Encrypted, persistent, reliable delivery)     │
│           │                                                 │
│  ┌────────▼───────────────────────────────────────┐        │
│  │  Central Collection Point (Load Balancer)      │        │
│  │  • High availability                           │        │
│  │  • Queue buffering                             │        │
│  │  • Rate limiting                               │        │
│  └────────┬───────────────────────────────────────┘        │
│                                                               │
└───────────────────────────┬───────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────┐
│              PROCESSING & ANALYSIS LAYER                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────┐           │
│  │  Wazuh Manager (Central Processing)          │           │
│  │  • IDS Rule Engine                           │           │
│  │  • Log parsing & enrichment                  │           │
│  │  • Alert generation                          │           │
│  │  • Threat intelligence integration           │           │
│  │  • Active response execution                 │           │
│  └──────────────┬───────────────────────────────┘           │
│                 │                                           │
│  ┌──────────────▼───────────────────────────────┐           │
│  │  Logstash (Log Processing Pipeline)          │           │
│  │  • Parsing, filtering, transforming          │           │
│  │  • Geographic IP enrichment                  │           │
│  │  • Custom field extraction                   │           │
│  │  • Vulnerability data correlation            │           │
│  └──────────────┬───────────────────────────────┘           │
│                 │                                           │
│  ┌──────────────▼───────────────────────────────┐           │
│  │  Elasticsearch (Full-Text Search Index)      │           │
│  │  • Distributed cluster (3-10 nodes)          │           │
│  │  • Real-time indexing                        │           │
│  │  • Complex aggregations                      │           │
│  │  • Fast retrieval (<2 seconds)               │           │
│  └──────────────┬───────────────────────────────┘           │
│                                                              │
└─────────────────┬──────────────────────────────────────────┘
                  │
┌─────────────────▼──────────────────────────────────────────┐
│              VISUALIZATIONS & ALERTING LAYER              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────┐           │
│  │  Kibana Dashboards                       │           │
│  │  • Security events timeline               │           │
│  │  • Top threats & attacker IPs            │           │
│  │  • System health & performance           │           │
│  │  • Compliance status reports             │           │
│  │  • Custom investigation views            │           │
│  └──────────────┬───────────────────────────┘           │
│                 │                                       │
│  ┌──────────────▼───────────────────────────┐           │
│  │  Wazuh Security Dashboard                │           │
│  │  • Alert severity metrics                │           │
│  │  • Agent status                          │           │
│  │  • Rule statistics                       │           │
│  │  • Compliance tracking                   │           │
│  └──────────────┬───────────────────────────┘           │
│                 │                                       │
│  ┌──────────────▼───────────────────────────┐           │
│  │  Alert Routing & Notifications           │           │
│  │  • Slack → real-time alerts              │           │
│  │  • PagerDuty → incident escalation       │           │
│  │  • Email → daily summaries               │           │
│  │  • Webhooks → custom integrations        │           │
│  │  • SOAR platform → automated response    │           │
│  └──────────────────────────────────────────┘           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Physical Deployment Architecture

### On-Premises Deployment

```
┌─ Management Network
│
├─ Monitoring Server (Physical or VM)
│  • 16-32 GB RAM
│  ├─ Wazuh Manager
│  ├─ Elasticsearch Node
│  ├─ Kibana
│  └─ SSL Certificate Authority
│
├─ Data Storage (SAN or NAS)
│  • RAID-10 SSD
│  • 500GB-5TB depending on retention
│  • Backup replication to secondary site
│
├─ Load Balancer (HA)
│  • Active-Passive failover
│  • Health checks every 10 seconds
│  • 99.99% availability target
│
└─ Firewall Rules
   • Agents: Port 1514 (UDP) or 1515 (TCP)
   • API: Port 55000 (TLS)
   • Elasticsearch: Port 9300 (cluster)
   • Kibana: Port 5601 (HTTP)
```

### Cloud-Native Deployment (AWS Example)

```
AWS VPC
├─ Private Subnet A (us-east-1a)
│  ├─ Wazuh Manager (t3.xlarge)
│  ├─ Elasticsearch Node 1 (m5.2xlarge)
│  └─ NAT Gateway
│
├─ Private Subnet B (us-east-1b)
│  ├─ Elasticsearch Node 2 (m5.2xlarge)
│  └─ Security Group Rules
│
├─ Private Subnet C (us-east-1c)
│  ├─ Elasticsearch Node 3 (m5.2xlarge)
│  └─ Kibana (t3.large)
│
├─ Application Load Balancer
│  ├─ Listener: 1514 → Manager
│  ├─ Listener: 5601 → Kibana
│  └─ Auto-scaling based on load
│
├─ RDS (Optional Postgres for audit logs)
│
├─ S3 Buckets
│  ├─ Elasticsearch snapshots
│  ├─ Backup/archives
│  └─ Long-term retention (Glacier)
│
└─ CloudWatch Monitoring
   ├─ Manager health checks
   ├─ Elasticsearch cluster status
   └─ Memory/CPU/Disk utilization
```

### Kubernetes Deployment

```
Kubernetes Cluster
│
├─ Namespace: security
│
├─ Wazuh Manager (Deployment)
│  ├─ 2+ replicas for HA
│  ├─ PVC: 100GB storage
│  ├─ Service: LoadBalancer
│  ├─ ConfigMap: Rules
│  └─ Secret: SSL certs, API keys
│
├─ Elasticsearch (StatefulSet)
│  ├─ 3+ replicas
│  ├─ PVC per node: 500GB
│  ├─ Service: Headless
│  ├─ Persistent Volumes
│  └─ Anti-Affinity rules
│
├─ Kibana (Deployment)
│  ├─ 2+ replicas
│  ├─ Service: ClusterIP
│  └─ Ingress: HTTPS with cert
│
├─ Falco (DaemonSet)
│  ├─ Runs on every node
│  ├─ Host PID namespace
│  ├─ eBPF kernel module
│  └─ Output to Logstash
│
├─ Filebeat (DaemonSet)
│  ├─ Collects container logs
│  ├─ Kubernetes metadata enrichment
│  └─ Output to Logstash
│
├─ Logstash (StatefulSet)
│  ├─ 2+ replicas
│  ├─ Horizontal Pod Autoscaler
│  └─ Output to Elasticsearch
│
└─ Network Policies
   ├─ Ingress restricted by source IP
   ├─ Egress to external logging permitted
   └─ Inter-pod communication rules
```

---

## High Availability Architecture

### Manager High Availability

```
┌──────────────────────────────────────────┐
│  External Load Balancer (DNS: wazuh.example.com)
├──────────────────────────────────────────┤
│         • Port 1514 → Manager 1/2        │
│         • Health checks every 10 sec     │
│         • Failover < 30 seconds          │
└──────────────────────────────────────────┘
         │          │
    ┌────▼──┐   ┌──▼─────┐
    │Manager│   │Manager  │
    │  1    │   │  2      │
    │       │   │         │
    │Shared │   │Shared   │
    │────────────────────
    │  Elasticsearch Cluster
    │  (3+ nodes, quorum-based)
    └─────────────────────┘
```

### Data Storage High Availability

```
┌─ Primary Elasticsearch Cluster (Production)
│  ├─ Node 1: us-east-1a
│  ├─ Node 2: us-east-1b
│  ├─ Node 3: us-east-1c
│  └─ Shards: 5, Replicas: 2 (redundancy 3x)
│
├─ Backup Strategy
│  ├─ Hourly snapshots to S3
│  ├─ 7-day retention (hot)
│  ├─ 90-day retention (cold)
│  └─ Cross-region replication
│
└─ Disaster Recovery
   ├─ RTO: 1 hour
   ├─ RPO: 30 minutes
   └─ Tested quarterly
```

---

## Network Architecture

### Firewall Rules

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| Agents | Manager | 1514 | UDP/TCP | Agent data |
| Manager | Elasticsearch | 9300 | TCP | Cluster comm |
| Elasticsearch | Manager | 9300 | TCP | Cluster comm |
| Kibana | Elasticsearch | 9200 | TCP | Query data |
| External | Kibana | 443 | HTTPS | Dashboard access |
| Agents | NTP Server | 123 | UDP | Time sync |
| Manager | DNS | 53 | UDP | Name resolution |

### VPC Security Groups

```
Manager Security Group:
  • Inbound 1514/TCP,UDP from Agents CIDR
  • Inbound 55000/TCP from Admin CIDR (API)
  • Inbound 9300/TCP from Elasticsearch CIDR
  • Outbound: All

Elasticsearch Security Group:
  • Inbound 9200/TCP from Manager, Kibana
  • Inbound 9300/TCP from Elasticsearch nodes
  • Outbound: All (for snapshots)

Kibana Security Group:
  • Inbound 443/TCP from Admin CIDR
  • Inbound 5601/TCP from LB
  • Outbound to Elasticsearch
```

---

## Integration Points

### Incident Response Integration

```
Detection (Wazuh)
    ↓
Alert Enrichment
    ↓
Severity Assessment
    ↓
Notification (PagerDuty)
    ↓
SOAR Playbook
    ↓
Automated Response
    ├─ Isolate host
    ├─ Kill process
    ├─ Collect evidence
    └─ Create ticket
```

### DevOps Pipeline Integration

```
Git Push
    ↓
CI Pipeline (GitLab CI/Jenkins)
    ├─ Run Trivy container scan
    ├─ Check vulnerability DB
    ├─ Block if critical found
    └─ Deploy if approved
    ↓
Environment Deployment
    ├─ Falco starts monitoring
    ├─ Agents begin collection
    └─ Alerts generated
```

---

## Backup & Disaster Recovery

### Backup Strategyegy

```
RTO: 4 hours | RPO: 1 hour

┌─ Real-time replication
│  • Elasticsearch replicas across zones
│  ├─ Hourly snapshots to S3
│  ├─ Cross-region replication
│  └─ 7-day hot retention
│
├─ Daily full backups
│  ├─ Manager configuration
│  ├─ Rules and decoders
│  ├─ Agent certificates
│  └─ 30-day retention
│
└─ Compliance archival
   ├─ Audit logs immutable
   ├─ 7-year retention (regulatory)
   └─ WORM storage
```

---

## Monitoring the Security Stack

### What to Monitor

| Component | Metric | Threshold | Action |
|-----------|--------|-----------|--------|
| **Manager** | CPU | > 80% | Add resources |
| | Memory | > 85% | Investigate, scale |
| | Disk | > 90% | Archive data |
| **Elasticsearch** | Cluster Health | Not Green | Investigate |
| | Heap Usage | > 85% | Upgrade nodes |
| | Shard Allocation | Unassigned | Fix immediately |
| **Kibana** | Response Time | > 2 sec | Optimize queries |
| | Errors | > 0 | Investigate |
| **Agents** | Disconnected | > 5% | Check network |
| | Failed Auth | Increasing | Security review |

---

## Capacity Planning

### Storage Calculation

```
Daily Logs = (Events per second) × 86,400 seconds × (avg bytes per event)

Example:
10K events/sec × 86,400 × 1 KB = 864 GB/day
× 30 days retention = 25.9 TB needed
× 2 (Elasticsearch replicas) = 51.8 TB
```

### Growth Projection

| Year | Agents | Daily Volume | Storage Needed |
|------|--------|--------------|----------------|
| 1 | 500 | 100 GB | 3 TB |
| 2 | 1,500 | 300 GB | 9 TB |
| 3 | 3,000 | 600 GB | 18 TB |
