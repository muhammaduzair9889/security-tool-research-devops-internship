# Deployment Architecture Specification

## Overview

This document provides detailed architectural specifications for deploying the recommended security infrastructure stack. The architecture encompasses intrusion detection, vulnerability assessment, and centralized logging components in an integrated, scalable, and highly available deployment.

---

## Architecture Principles

### Design Principles

1. **High Availability** - No single point of failure for critical security functions
2. **Scalability** - Horizontal scaling capability across all components
3. **Security Isolation** - Network segmentation and least privilege access
4. **Operational Simplicity** - Balance sophistication with maintainability
5. **Cost Optimization** - Efficient resource utilization without compromising capability

### Deployment Standards

- Infrastructure as Code (IaC) for all component deployments
- Configuration management via version control
- Automated backup and disaster recovery procedures
- Comprehensive monitoring and alerting
- Documentation of all architectural decisions

---

## Logical Architecture

### System Components

#### Intrusion Detection Layer (Wazuh)

**Manager Cluster**
- 3-node manager cluster for high availability
- Load balancer frontend for agent connections
- Shared Elasticsearch backend for event storage
- Filebeat for log forwarding

**Agent Deployment**
- Agents on all monitored systems
- Agent configuration managed centrally
- Automatic agent registration and key management
- Agent health monitoring

**Integration Points**
- Elasticsearch for data storage
- Kibana for visualization
- External SIEM via syslog or API
- Ticketing system integration (Jira, ServiceNow)
- Email and Slack for alerting

#### Vulnerability Assessment Layer

**Trivy Scanning Infrastructure**
- Container registry integration (Harbor, ECR, ACR)
- CI/CD pipeline integration (Jenkins, GitLab, GitHub Actions)
- Scheduled scanning via cron or orchestrator
- Results aggregation and reporting

**Grype Deployment**
- Source code repository integration
- SBOM generation pipeline
- Continuous dependency monitoring
- Vulnerability tracking database

**OpenVAS Infrastructure**
- OpenVAS Manager server
- PostgreSQL database for configuration
- Scanner nodes for distributed scanning
- Web interface for management and reporting

#### Centralized Logging Layer (Grafana Loki)

**Loki Cluster**
- 3-node Loki cluster for high availability
- Object storage backend (S3, GCS, MinIO)
- Distributed query execution
- Log retention policies

**Log Collection**
- Promtail agents on all systems
- Fluentd/Fluent Bit for special cases
- Direct integration with Kubernetes
- Syslog collectors for network devices

**Visualization and Analysis**
- Grafana dashboards
- LogQL query interface
- Alerting rules via Alertmanager
- Integration with Wazuh for correlation

---

## Physical Deployment Patterns

### Small Organization Deployment (50 GB logs daily)

**Infrastructure Requirements**

Wazuh Infrastructure:
- 1 manager node: 4 vCPU, 8 GB RAM, 100 GB storage
- 1 Elasticsearch node: 4 vCPU, 16 GB RAM, 500 GB SSD
- Up to 50 agents

Vulnerability Scanning:
- 1 Trivy instance: 2 vCPU, 4 GB RAM, 50 GB storage
- 1 Grype instance: 2 vCPU, 2 GB RAM, 20 GB storage
- 1 OpenVAS server: 4 vCPU, 8 GB RAM, 200 GB storage

Grafana Loki:
- 1 Loki node: 4 vCPU, 8 GB RAM, 100 GB storage
- Object storage: 2 TB capacity
- 1 Grafana instance: 2 vCPU, 4 GB RAM, 50 GB storage

Total Infrastructure: 6 servers/VMs

**Network Architecture**
- Management network: 10.0.1.0/24
- Production monitoring: 10.0.2.0/24
- Storage network: 10.0.3.0/24
- Firewall rules between segments

### Medium Organization Deployment (500 GB logs daily)

**Infrastructure Requirements**

Wazuh Infrastructure:
- 3 manager nodes (HA cluster): 8 vCPU, 16 GB RAM each, 200 GB storage
- 1 load balancer: 2 vCPU, 4 GB RAM
- 5 Elasticsearch nodes: 8 vCPU, 32 GB RAM each, 2 TB SSD
- Up to 500 agents

Vulnerability Scanning:
- 2 Trivy instances (redundant): 4 vCPU, 8 GB RAM each, 100 GB storage
- 2 Grype instances: 4 vCPU, 4 GB RAM each, 50 GB storage
- 2 OpenVAS servers (distributed): 8 vCPU, 16 GB RAM each, 500 GB storage

Grafana Loki:
- 3 Loki nodes (cluster): 8 vCPU, 16 GB RAM each, 200 GB storage
- Object storage: 20 TB capacity
- 2 Grafana instances (HA): 4 vCPU, 8 GB RAM each, 100 GB storage
- 3 Promtail aggregators: 2 vCPU, 4 GB RAM each

Total Infrastructure: 21 servers/VMs

**Network Architecture**
- Management VLAN: 10.0.1.0/24
- Security monitoring VLAN: 10.0.2.0/24
- Production monitoring VLAN: 10.0.3.0/24
- Storage VLAN: 10.0.4.0/24
- DMZ for external access: 10.0.5.0/24
- Firewall and IDS between VLANs

### Large Organization Deployment (5 TB logs daily)

**Infrastructure Requirements**

Wazuh Infrastructure:
- 5 manager nodes (HA cluster): 16 vCPU, 32 GB RAM each, 500 GB storage
- 2 load balancers (active-passive): 4 vCPU, 8 GB RAM each
- 15 Elasticsearch nodes: 16 vCPU, 64 GB RAM each, 5 TB SSD
- 3 Kibana instances: 8 vCPU, 16 GB RAM each
- Up to 5000 agents

Vulnerability Scanning:
- 5 Trivy instances (distributed): 8 vCPU, 16 GB RAM each, 500 GB storage
- 3 Grype instances: 8 vCPU, 8 GB RAM each, 200 GB storage
- 5 OpenVAS servers (distributed): 16 vCPU, 32 GB RAM each, 1 TB storage

Grafana Loki:
- 9 Loki nodes (3 read, 3 write, 3 backend): 16 vCPU, 32 GB RAM each, 500 GB storage
- Object storage: 200 TB capacity
- 3 Grafana instances (load balanced): 8 vCPU, 16 GB RAM each, 200 GB storage
- 10 Promtail aggregators: 4 vCPU, 8 GB RAM each

Total Infrastructure: 58 servers/VMs

**Network Architecture**
- Multiple data centers for geographic distribution
- Dedicated security monitoring network
- High-bandwidth inter-datacenter links (10+ Gbps)
- Redundant network paths
- Network segmentation by function and security zone

---

## High Availability Configuration

### Wazuh High Availability

**Manager Cluster Architecture**

Cluster Configuration:
- 3 or 5 manager nodes in active-active cluster
- Shared Elasticsearch backend for state synchronization
- Load balancer distributing agent connections
- Automatic failover within 30 seconds

Load Balancer Configuration:
- TCP load balancing on port 1514 (agent registration)
- Health checks every 10 seconds
- Automatic node removal on failure
- Session persistence for agent connections

**Elasticsearch Cluster**

Cluster Configuration:
- Minimum 3 master-eligible nodes (5 recommended)
- Dedicated master nodes for large deployments
- Data nodes with hot/warm architecture
- Coordinating-only nodes for query distribution

Shard Configuration:
- Primary shards: 5 per index
- Replica shards: 1 or 2 depending on criticality
- Index lifecycle management for retention
- Snapshot backups every 6 hours

### Grafana Loki High Availability

**Loki Microservices Mode**

Component Distribution:
- Distributor: 3+ instances for log ingestion
- Ingester: 3+ instances for log storage
- Querier: 3+ instances for query execution
- Query Frontend: 2+ instances for caching
- Compactor: 1 instance for compaction
- Ruler: 2+ instances for alerting

Object Storage:
- S3-compatible object storage with versioning
- Replication factor of 3 for durability
- Lifecycle policies for cost optimization
- Cross-region replication for disaster recovery

**Grafana High Availability**

Configuration:
- Multiple Grafana instances behind load balancer
- Shared database (PostgreSQL or MySQL)
- Session storage in Redis for state
- Automated health checks and failover

### Vulnerability Scanning Redundancy

**Trivy Redundancy**

Deployment:
- Multiple Trivy instances for parallel scanning
- Load balancing for CI/CD integrations
- Cached vulnerability database on shared storage
- Automatic failover to backup instances

**OpenVAS High Availability**

Configuration:
- Multiple scanner nodes
- Shared PostgreSQL database (HA configured)
- Task distribution and load balancing
- Manager failover with keepalived or similar

---

## Network Architecture

### Network Segmentation

**Security Zones**

Management Zone:
- Access to all management interfaces
- Restricted to jump boxes and admin workstations
- Multi-factor authentication required
- Full audit logging

Monitoring Zone:
- Wazuh managers and Elasticsearch
- Grafana and Loki infrastructure
- One-way traffic from monitored systems
- No direct internet access

Scanning Zone:
- Vulnerability scanning infrastructure
- Access to all networks for scanning
- Rate-limited and scheduled access
- Results export only to management zone

Data Storage Zone:
- Elasticsearch data nodes
- Loki object storage
- Backup infrastructure
- Encrypted at rest and in transit

### Firewall Rules

**Inbound to Management Zone**
- SSH (22/tcp) from jump boxes only
- HTTPS (443/tcp) from admin network
- All traffic logged and monitored

**Inbound to Monitoring Zone**
- Wazuh agent traffic (1514/tcp, 1515/tcp) from all monitored systems
- Syslog (514/tcp, 514/udp) from network devices
- API access (9200/tcp) from authorized systems only
- Promtail data (3100/tcp) from all systems

**Outbound from Monitoring Zone**
- NTP (123/udp) for time synchronization
- DNS (53/tcp, 53/udp) for name resolution
- HTTPS (443/tcp) for threat intelligence feeds
- Email (587/tcp) for alerting

**Inter-Zone Communication**
- Elasticsearch cluster communication (9300/tcp)
- Loki components (internal ports)
- Database replication and synchronization
- Backup and replication traffic

---

## Storage Architecture

### Data Storage Requirements

**Elasticsearch Storage**

Index Structure:
- Daily indices for Wazuh events
- Weekly indices for vulnerability scan results
- Index aliases for easy querying
- Index lifecycle management

Storage Tiers:
- Hot tier: NVMe SSD, 7-day retention
- Warm tier: SATA SSD, 30-day retention
- Cold tier: HDD or object storage, 365-day retention
- Frozen tier: Object storage, long-term archival

Capacity Planning:
- Small org: 500 GB hot, 2 TB warm, 10 TB cold
- Medium org: 5 TB hot, 20 TB warm, 100 TB cold
- Large org: 50 TB hot, 200 TB warm, 1 PB cold

**Loki Object Storage**

Storage Layout:
- Chunks in object storage
- Index in database or object storage
- Compaction reduces storage by 10x
- Retention policies delete old data

Capacity Planning:
- Small org: 2 TB total
- Medium org: 20 TB total
- Large org: 200 TB total

**Backup Storage**

Backup Strategy:
- Full backup weekly
- Incremental backup daily
- Configuration backup after changes
- Offsite replication for disaster recovery

Retention:
- Daily backups: 30 days
- Weekly backups: 90 days
- Monthly backups: 12 months
- Yearly backups: 7 years (compliance)

---

## Database Architecture

### PostgreSQL for OpenVAS

**Configuration**

Deployment:
- PostgreSQL 14 or later
- Primary-replica configuration for HA
- Automated failover with Patroni
- Connection pooling with PgBouncer

Performance Tuning:
- shared_buffers: 25% of RAM
- effective_cache_size: 50% of RAM
- work_mem: 50-100 MB per connection
- maintenance_work_mem: 1-2 GB

**Backup and Recovery**

Backup Strategy:
- Continuous archiving (WAL shipping)
- Daily base backups
- Point-in-time recovery capability
- Backup verification and testing

### Shared Database for Grafana

**Configuration**

Deployment:
- MySQL or PostgreSQL
- Master-replica for read scaling
- Redis for session storage
- Regular backup and testing

---

## Monitoring and Alerting Design

### Infrastructure Monitoring

**System Metrics**

Collection:
- Prometheus for metrics collection
- Node Exporter on all systems
- Custom exporters for applications
- 15-second scrape interval

Metrics:
- CPU, memory, disk, network utilization
- Process status and performance
- Storage capacity and IOPS
- Network throughput and packet loss

**Application Monitoring**

Wazuh Monitoring:
- Manager and agent status
- Event processing rate
- Queue sizes and backlog
- Alert generation rate

Loki Monitoring:
- Ingestion rate
- Query performance
- Storage utilization
- Component health status

Vulnerability Scanning:
- Scan completion rate
- Scan duration and performance
- Database update status
- Finding generation rate

### Alerting Configuration

**Critical Alerts** (Immediate notification)
- Service downtime or unavailability
- Cluster node failure
- Disk space >90% full
- Security incident detection
- Backup failure

**Warning Alerts** (1-hour response)
- Performance degradation
- Elevated error rates
- Capacity approaching limits
- Configuration drift detected
- Certificate expiration <30 days

**Informational Alerts**
- Scheduled maintenance completion
- Backup completion
- System health reports
- Capacity planning reports

---

## Security Hardening

### Operating System Hardening

**Security Configuration**

- Minimal OS installation
- SELinux or AppArmor enabled
- Automatic security updates
- Host-based firewall (iptables/nftables)
- Fail2ban or similar for brute force protection

**Access Control**

- SSH key-only authentication
- Sudo access with logging
- PAM configuration for strong passwords
- Regular access review and removal

### Application Security

**Wazuh Security**

- TLS encryption for all communication
- Certificate-based authentication
- API key rotation every 90 days
- Regular rule and configuration review

**Loki Security**

- Authentication required for all access
- TLS encryption for data in transit
- Object storage encryption at rest
- Audit logging enabled

**Network Security**

- VPN for remote access
- Jump box for administrative access
- Network intrusion detection
- Regular vulnerability scanning

---

## Disaster Recovery

### Recovery Objectives

**Recovery Time Objective (RTO)**
- Critical systems: 4 hours
- Non-critical systems: 24 hours
- Full environment: 48 hours

**Recovery Point Objective (RPO)**
- Configuration: 24 hours
- Security events: 1 hour
- Logs: 24 hours

### Recovery Procedures

**Manager Node Failure**

1. Verify node failure through monitoring
2. Load balancer automatically removes failed node
3. Remaining nodes handle load
4. Provision replacement node
5. Restore configuration from backup
6. Re-join to cluster

**Data Loss Scenario**

1. Identify extent of data loss
2. Stop ingestion to corrupted indices
3. Restore from most recent backup
4. Replay logs from retention buffer
5. Verify data integrity
6. Resume normal operations

**Complete Site Failure**

1. Activate disaster recovery site
2. Restore infrastructure from IaC
3. Restore data from offsite backups
4. Update DNS and load balancer configuration
5. Verify all systems operational
6. Redirect traffic to DR site

---

## Deployment Checklist

### Pre-Deployment

- [ ] Infrastructure provisioned and validated
- [ ] Network configuration complete
- [ ] Security hardening applied
- [ ] Backup infrastructure ready
- [ ] Monitoring infrastructure deployed

### Deployment

- [ ] Install and configure Wazuh managers
- [ ] Deploy Elasticsearch cluster
- [ ] Configure Wazuh agents
- [ ] Deploy Grafana Loki
- [ ] Configure Promtail agents
- [ ] Deploy vulnerability scanning tools
- [ ] Configure integrations

### Post-Deployment

- [ ] Validate all components operational
- [ ] Test high availability failover
- [ ] Verify backup and recovery
- [ ] Configure alerting rules
- [ ] Train operations team
- [ ] Document as-built configuration

---

## Conclusion

This deployment architecture provides a scalable, highly available, and secure foundation for enterprise security monitoring. The architecture balances operational complexity with capability, ensuring reliable protection while maintaining cost efficiency.

Regular review and optimization of the architecture ensures continued alignment with organizational needs and industry best practices.
